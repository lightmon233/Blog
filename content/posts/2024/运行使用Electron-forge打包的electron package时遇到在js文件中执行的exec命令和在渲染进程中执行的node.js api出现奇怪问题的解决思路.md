---
title: "运行使用Electron-forge打包的electron package时遇到在js文件中执行的exec命令和在渲染进程中执行的node.js api出现奇怪问题的解决思路"
date: 2024-10-10T23:30:31+08:00
draft: false
description: ""
---
{{< katex >}}


# js文件中执行的exec命令出错

很可能是项目中使用了一些非`html`, `css`, `js`的源文件，比如用了`Makefile`来编译了`cpp`代码，或者执行的`exec`命令为`cp dir/something.cpp`之类的文件操作命令。

可以使用修改`forge.config.js`文件配置的方式，使得`npm run make`的时候自动把`Makefile`等`exec`命令中用到的文件和目录复制到打包后的根目录中。

具体来说，可以给`forge.config.js`文件的`module.exports/packagerConfig`下新增如下配置：

```js
afterExtract: [
  (extractPath, electronVersion, platform, arch, done) => {
    // Copy the Makefile to the build directory
    var makefile = path.join(__dirname, 'Makefile');
    var backup = path.join(__dirname, 'backup');
    var dest = extractPath;
    fs.copyFileSync(makefile, path.join(dest, 'Makefile'));
    // 把backup目录递归地复制到dest目录下面
    copyDir(backup, path.join(dest, 'backup'));
    done();
  }
]
```

其中，`copyDir`函数的定义为：
```js
function copyDir(src, dest) {
  fs.mkdirSync(dest, { recursive: true }); // 创建目标目录

  // 递归遍历源目录
  fs.readdirSync(src).forEach(file => {
    const srcPath = path.join(src, file);
    const destPath = path.join(dest, file);

    // 判断是文件还是目录
    const stat = fs.statSync(srcPath);
    if (stat.isDirectory()) {
      copyDir(srcPath, destPath); // 递归复制子目录
    } else {
      fs.copyFileSync(srcPath, destPath); // 复制文件
    }
  });
}
```

# 渲染进程中执行的node.js api出错

可能是打包后的浏览器内核安全设置更高？

建议electron项目中还是尽量前后端分离比较好，在主进程中用`node.js`的库，即各种`require`，在渲染进程（浏览器进程）中就用web页面的库，即各种`import`和`export`。

我把`renderer.js`中的`fs`模块的操作都删了，把调用的`fs`模块函数都转移到了`main.js`里面去执行，这个错误就消失了。

比如原本在`renderer.js`中是这样：

```js
document.getElementById('run-compare').addEventListener('click', () => {
  // 获取code1和code2的内容
  var code1Content = editor1.state.doc.toString(); 
  var code2Content = editor2.state.doc.toString();

  // 获取项目根目录
  rootPath = __dirname;

  // 创建文件路径
  const filePath1 = path.join(rootPath, 'code1.cpp');
  const filePath2 = path.join(rootPath, 'code2.cpp');

  // 写入文件
  fs.writeFile(filePath1, code1Content, (err) => {
    if (err) {
      console.error(err);
      return;
    }
    console.log('code1写入成功');
  });

  fs.writeFile(filePath2, code2Content, (err) => {
    if (err) {
      console.error(err);
      return;
    }
    console.log('code2写入成功');
  });

  ipcRenderer.send('run-compare');
});
```

现在我就改成了这样：
```js
document.getElementById('run-compare').addEventListener('click', () => {
  // 获取code1和code2的内容
  var code1Content = editor1.state.doc.toString(); 
  var code2Content = editor2.state.doc.toString();

  // 获取项目根目录
  rootPath = __dirname;

  // 创建文件路径
  const filePath1 = path.join(rootPath, 'code1.cpp');
  const filePath2 = path.join(rootPath, 'code2.cpp');

  // 发送消息给主进程
  ipcRenderer.send('compare-codes', { code1: code1Content, code2: code2Content });

  ipcRenderer.send('run-compare');
});
```

然后在`main.js`中新增对`compare-codes`事件的监听：

```js
ipcMain.on('compare-codes', (event, data) => {
  const { code1, code2 } = data;
  fs.writeFileSync('code1.cpp', code1);
  fs.writeFileSync('code2.cpp', code2);
});
```

总之，以后用`node.js`写项目，最好还是要把前后端分离，不能在被某个`html`文件引用的`js`脚本中调用`node.js`中的库。

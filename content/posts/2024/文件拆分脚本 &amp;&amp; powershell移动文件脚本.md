---
title: "文件拆分脚本 &amp;&amp; powershell移动文件脚本"
date: 2024-06-17T21:42:13+08:00
draft: false
description: ""
---
{{< katex >}}


## 文件拆分脚本

每隔两行拆分成一个新文件。

```py
import os

with open('Main.java', 'r', encoding='UTF-8') as file:
    file_content = file.read()
    file_parts = file_content.split('\n\n')
    for i in range(len(file_parts)):
        fp = open(f'{i + 1}.txt', 'w')
        fp.close()
        for row in file_parts[i].splitlines():
            row = row[2:]
            print(row)
            with open(f'{i + 1}.txt', 'a') as new:
                new.write(row)
                new.write('\n')
        
```

## powershell移动文件脚本

把同名图片和文件放到同名文件夹底下。

```ps
# 获取当前目录的所有文件

$files = Get-ChildItem -Recurse -File

# 从files中删除当前脚本文件
\(files = \)files | Where-Object { \(_.FullName -ne \)MyInvocation.MyCommand.Path }

foreach (\(file in \)files) {
    # 获取文件的目录
    \(targetDir = \)file.BaseName
    if (Test-Path -Path $targetDir) {
        Move-Item -Path \(file.FullName -Destination \)targetDir
    }
    else {
        New-Item -ItemType Directory -Path $targetDir
        Move-Item -Path \(file.FullName -Destination \)targetDir
    }
}
```
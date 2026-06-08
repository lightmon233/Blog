---
title: "openresty配置为简单文件上传下载服务器"
date: 2024-11-11T11:36:01+08:00
draft: false
description: ""
categories: ["技术折腾"]
tags: ["openresty", "nginx", "server"]
---
{{< katex >}}


安装`resty.upload`模块
```bash
opm install ledgetech/lua-resty-upload
```

新建`/usr/local/openresty/nginx/lua/upload.lua`

内容如下：
```lua
local upload = require "resty.upload"
local cjson = require "cjson.safe"

local chunk_size = 4096
local form, err = upload:new(chunk_size)

if not form then
    ngx.log(ngx.ERR, "failed to create upload: ", err)
    ngx.exit(500)
end

local file
local file_name = ""
local success = false

-- 获取 URL 参数中的子文件夹路径（支持嵌套）
local folder = ngx.var.arg_folder
if not folder or folder == "" then
    folder = ""  -- 如果未提供 folder 参数，使用默认子文件夹
end

-- 构建文件存储的完整路径
local file_path = "/home/ubuntu/share/" .. folder .. "/"

-- 递归创建嵌套目录（确保子文件夹路径存在）
local mkdir_command = "mkdir -p " .. file_path
os.execute(mkdir_command)

form:set_timeout(1000)  -- 设置超时时间

while true do
    local typ, res, err = form:read()
    if not typ then
        ngx.say(cjson.encode({success = false, msg = "failed to read: " .. err}))
        return
    end

    if typ == "header" then
        if res[1] == "Content-Disposition" then
            file_name = res[2]:match('filename="([^"]+)"')
            if file_name then
                file = io.open(file_path .. file_name, "w+")
                if not file then
                    ngx.say(cjson.encode({success = false, msg = "failed to open file"}))
                    return
                end
            end
        end

    elseif typ == "body" then
        if file then
            file:write(res)
        end

    elseif typ == "part_end" then
        if file then
            file:close()
            file = nil
            success = true
        end

    elseif typ == "eof" then
        break
    end
end

ngx.say(cjson.encode({success = success, file = file_name, path = file_path}))
```

修改`/usr/local/openresty/nginx/conf/nginx.conf`
```bash
user ubuntu;

events {}

http {
        server {
        listen 80;
        server_name localhost;

        # 文件上传接口
        location /upload/ {
            content_by_lua_file /usr/local/openresty/nginx/lua/upload.lua;
        }

        # 文件下载接口
        location /download/ {
            alias /home/ubuntu/share/;
            autoindex on;   # 开启目录浏览
            autoindex_exact_size off; # 使用MB, GB作为文件大小单位
            autoindex_localtime on; # 显示本机
        }
    }
}
```

设置`/home/ubuntu/share`目录权限
```bash
sudo chown -R www-data:www-data /home/ubuntu/share
sudo chmod -R 755 /home/ubuntu/share
```

修改nginx运行时用户为目录owner
在`nginx.conf`中首行写入：
```bash
user 用户名 用户组; # 这里的用户名和组就是文件夹的
```

可以通过命令：
```bash
ps -aux | grep nginx
```
查看nginx worker用户。

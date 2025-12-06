# 使用

既然我们要“具体怎么用”，那我们就抛开空泛的概念，直接进入**实战模式**。

鉴于你之前提到正在做**AI 项目（Python/vllm/Dify）**，且使用的是 **Linux**（或打算在 ThinkPad 上装 Linux），我将以 **Linux (Ubuntu/Debian)** 环境为标准，为你梳理一套**从安装到高阶管理**的标准工作流。

---

### 第一步：安装与验证

在 Linux 上，尽量不要下载源码编译（除非你是为了魔改内核），使用包管理器是最稳妥的。

Bash

```
# 1. 更新软件源
sudo apt update

# 2. 安装 Nginx
sudo apt install nginx

# 3. 验证是否安装成功（看到版本号即成功）
nginx -v

# 4. 检查服务状态
systemctl status nginx
```

_此时，你在浏览器输入服务器 IP（或 `localhost`），应该能看到 "Welcome to nginx!" 的默认页面。_

---

### 第二步：掌握“驾驶舱”指令

作为管理员，你只需要记住这 4 个核心指令。请注意，**Reload** 比 Restart 更重要。

|**指令**|**作用**|**什么时候用？**|**备注**|
|---|---|---|---|
|`nginx -t`|**体检** (Test)|**每次**修改配置文件后，必须先运行它。|它会告诉你配置语法对不对，避免重启后挂掉。|
|`nginx -s reload`|**热重载**|修改配置后，让配置生效。|**0 停机时间**。它会启动新 Worker，平滑杀掉旧 Worker。|
|`systemctl restart nginx`|**硬重启**|只有在修改了系统级配置（如用户组、文件句柄限制）时才用。|会有短暂的服务中断。|
|`systemctl stop nginx`|**停止**|维护或卸载时。|-|

---

### 第三步：理解配置文件架构（地图）

Nginx 的配置并不杂乱，它有严格的层级。Linux 下的主配置文件在 `/etc/nginx/nginx.conf`。

最佳实践架构（Ubuntu/Debian 风格）：

不要把所有配置都塞进主文件！我们要利用 include 机制。

- `/etc/nginx/nginx.conf`: **全局配置**（一般不动）。
- `/etc/nginx/conf.d/*.conf`: **自定义配置**（放这里）。
- _(推荐)_ `/etc/nginx/sites-available/`: 存放**所有**站点的配置文件。
上
---

### 第四步：三个核心场景实战

我们要解决你目前业务中可能遇到的三个具体问题：**部署前端页面**、**代理 AI 后端接口**、**负载均衡**。

#### 场景 1：托管静态网站（比如你的“喜霖荟”前端）

假设你的前端代码（HTML/CSS/JS）放在 `/var/www/xilinhui/html`。

1. 创建一个配置文件：`sudo vim /etc/nginx/conf.d/xilinhui.conf`
2. 写入内容：

Nginx

```
server {
    listen 80;
    server_name www.xilinhui.com; # 你的域名

    # 根目录指向你的项目文件夹
    root /var/www/xilinhui/html;
    index index.html;

    location / {
        # 这一行非常重要！
        # 如果是 Vue/React 单页应用(SPA)，必须加 try_files。
        # 它的意思是：先找文件，找不到找目录，再找不到就返回 index.html (交给前端路由处理)
        try_files $uri $uri/ /index.html;
    }
}
```

#### 场景 2：反向代理 AI 服务（Python/vllm/Dify）

假设你在服务器 `127.0.0.1:8000` 上运行了一个 Python 的 AI 服务（比如 FastAPI 或 vllm），你想通过 `api.xilinhui.com` 访问它，而不是暴露 8000 端口。

Nginx

```
server {
    listen 80;
    server_name api.xilinhui.com;

    location / {
        # 转发请求给本地的 Python 服务
        proxy_pass http://127.0.0.1:8000;

        # 标准头部设置（由底向上的原理：必须告诉后端真实的请求者是谁）
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; # 告诉 Python 真实的客户端 IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # 如果是流式输出（如 ChatGPT 的打字机效果），必须关闭缓冲
        proxy_buffering off; 
    }
}
```

#### 场景 3：负载均衡（扩容）

假设你的 AI 业务火了，一台 GPU 跑不动，你在 8001, 8002, 8003 起了三个 vllm 实例。

Nginx

```
# 定义一个上游服务器组
upstream ai_nodes {
    # 默认是轮询策略 (Round Robin)
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003 weight=2; # weight=2 表示这台机器性能好，多分配点请求
}

server {
    listen 80;
    server_name api.xilinhui.com;

    location / {
        # 直接把请求甩给这个组，Nginx 自动分配
        proxy_pass http://ai_nodes;
    }
}
```

---

### 第五步：标准上线流程（SOP）

这是从开发到上线的标准动作：

1. **编写配置**：在 `/etc/nginx/conf.d/` 下新建 `my_project.conf`。
2. **语法检查**：运行 `sudo nginx -t`。如果报错，它会告诉你第几行错了。
3. **热加载**：运行 `sudo nginx -s reload`。
4. 查看日志：如果访问出错（比如 502 Bad Gateway 或 403 Forbidden），第一时间看错误日志：
    
    tail -f /var/log/nginx/error.log

---

### 常见“坑”与排错原理

1. **403 Forbidden**：
    
    - **原理**：Nginx 的 Worker 进程通常由用户 `www-data` 或 `nginx` 运行。如果你的网页文件放在 `/root/` 目录下，Nginx 根本没权限进去读取。
    - **解决**：把文件放在 `/var/www/` 下，或者修改目录权限。
        
2. **502 Bad Gateway**：
    
    - **原理**：Nginx 作为代理，它要把电话转接给后端（如 Python）。如果你没接通，说明后端挂了，或者端口写错了。
    - **解决**：检查你的 Python/vllm 服务是不是真的在运行。
        
3. **Client Body Too Large**：
    
    - **原理**：如果你要上传图片到服务器，Nginx 默认限制上传大小为 1MB。
    - **解决**：在 `http` 或 `server` 块中加入 `client_max_body_size 10M;`。

我可以为你做的下一步：

你的 ThinkPad 工程机如果要装 Linux 做开发机，我可以为你提供一份基于 WSL2 (Windows Subsystem for Linux) 或 双系统 的环境搭建指南，顺便教你如何用 Nginx 把你本地的 AI 模型通过局域网暴露给办公室的其他电脑使用。
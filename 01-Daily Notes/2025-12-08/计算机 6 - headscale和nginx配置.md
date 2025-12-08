# nginx 配置

确实，这份文件看起来“乱七八糟”是非常正常的。它现在是一个**大杂烩**，里面混合了：

1. **Debian/Ubuntu 的官方废话**（大段注释，教你做人）。
2. **被注释掉的功能模板**（PHP, .htaccess 等，你根本没用上）。
3. **默认的“欢迎”页面配置**（处理 `_` 域名）。
4. **你的业务配置**（Headscale 反向代理）。
5. **Certbot 自动插入的代码**（底部的 SSL 证书路径和 301 跳转）。

**这就好比你把“房屋使用说明书”、“装修合同”和“你的日记”全写在一张纸上了。**

为了以后的维护（以及你的心理健康），我们必须把它**拆分**并**清洗**。

---

### 改造方案：一分为二

根据 Linux 的最佳实践（我们在上一条对话中提到的 `sites-available` 管理哲学），我们应该把这个文件拆成两个：

1. **`default`**: 只负责兜底（比如别人直接通过 IP 访问你的服务器，或者访问了你没绑定的域名），显示一个 404 或者默认页。
2. **`headscale.conf`**: 专门负责 Headscale 的业务逻辑，清爽、无废话。

下面是清理后的代码，你可以直接复制替换。

---

#### 1. 新的 `/etc/nginx/sites-available/default` (纯净版)

保留这个文件的目的是为了**占位**。它告诉 Nginx：“如果有任何我不认识的域名来访问，或者有人直接用 IP 访问 80 端口，就由我来处理。”

```Nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # 下划线表示“任何没有专门配置server_name的域名”
    server_name _;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        # 如果找不到文件，返回 404
        try_files $uri $uri/ =404;
    }
}
```

---

#### 2. 新建 `/etc/nginx/sites-available/headscale.conf` (业务版)

这里我去掉了一切不需要的东西。

- **删除了 `root` 和 `index`**：你是反向代理 (`proxy_pass`)，流量直接转给后端 App 了，Nginx 不需要去硬盘里找 HTML 文件，留着那些配置不仅多余，还容易误导。
- **整理了 SSL**：Certbot 的配置保留，但排版整齐。

```Nginx
server {
    server_name headscale.williamlao.me;

    # 1. 核心业务：反向代理到 Headscale
    location / {
        # 你的后端地址
        proxy_pass http://38.57.234.225:8080;

        # 传递真实头部信息 (Standard Proxy Headers)
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 支持 (Headscale 必须)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # 建议加上：关闭缓冲，让控制流更顺畅
        proxy_buffering off;
    }

    # 2. SSL 配置 (由 Certbot 管理)
    listen [::]:443 ssl ipv6only=on; 
    listen 443 ssl; 
    
    # 证书路径 (原样保留)
    ssl_certificate /etc/letsencrypt/live/headscale.williamlao.me/fullchain.pem; 
    ssl_certificate_key /etc/letsencrypt/live/headscale.williamlao.me/privkey.pem; 
    include /etc/letsencrypt/options-ssl-nginx.conf; 
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; 
}

# 3. HTTP 强制跳转 HTTPS (由 Certbot 管理，优化格式)
server {
    if ($host = headscale.williamlao.me) {
        return 301 https://$host$request_uri;
    }

    listen 80;
    listen [::]:80;
    server_name headscale.williamlao.me;
    return 404; 
}
```

---

### 如何落地执行？

请按顺序在终端执行以下命令，完成这次“大扫除”：

**第一步：备份（以防万一）**

```Bash
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
```

**第二步：覆写 default 文件**

```Bash
# 使用 nano 或 vim 编辑 default，把上面第一段代码粘贴进去，覆盖原内容
sudo nano /etc/nginx/sites-available/default
```

**第三步：创建 Headscale 专属配置**

```Bash
# 新建文件
sudo nano /etc/nginx/sites-available/headscale.conf
# 把上面第二段代码粘贴进去
```

**第四步：启用新配置（创建软链接）**

```Bash
# 这里的逻辑是：把 available 里的文件链接到 enabled 里，就像创建快捷方式
sudo ln -s /etc/nginx/sites-available/headscale.conf /etc/nginx/sites-enabled/
```

**第五步：体检并重载**

```Bash
# 必须做！检查语法有没有错
sudo nginx -t

# 如果显示 OK，则平滑重启
sudo nginx -s reload
```

### 为什么之前的配置里会有 `root` 和 `try_files`？

你原本的配置里，`server` 块里既写了 `proxy_pass` (我是个代理)，又写了 `root /var/www/html` (我是个文件服务器)。

虽然 Nginx 足够聪明，看到 `proxy_pass` 就会忽略 `root`，但这在逻辑上是**精神分裂**的。一旦哪天你手误删了 `proxy_pass`，Nginx 就会突然把你服务器上的 HTML 源码暴露出来，这在安全上也是个隐患。

现在的配置把它们彻底分开了，逻辑清晰，看着也舒服。

# headsclae 配置

这是一个非常关键的节点。因为我们刚刚给 Headscale 穿上了一层“HTTPS 的外衣”（Nginx），如果 Headscale 自己不知道这回事，客户端连接时就会报错。

按以下步骤操作，确保一次成功：

### 第一步：检查 Headscale 自身的配置（必须做！）

在重启 Headscale 之前，你必须确保 Headscale 知道它的新身份是 `https://headscale.williamlao.me`，而不是原来的 IP 地址。

1. 编辑 Headscale 配置文件：

    ```Bash
    sudo nano /etc/headscale/config.yaml
    ```
    
2. 找到 `server_url` 这一行，**必须**修改为你的域名（带 https）：

    ```YAML
    # 之前可能是 http://38.57.234.225:8080
    # 现在改为：
    server_url: https://headscale.williamlao.me
    ```
    
    _注意：这是给客户端用的，告诉客户端认证服务器在哪里。_
    
3. 确保监听地址没变（配合你的 Nginx `proxy_pass`）：

    ```YAML
    listen_addr: 0.0.0.0:8080
    ```
    
    _(因为 Nginx 是转发到 8080 端口，所以这里保持 8080 不变，且不需要在 Headscale 里配置 SSL 证书，因为 Nginx 已经替它扛下了 SSL 的活)_。

### 第二步：重启 Headscale

修改完配置文件后，**必须重启** Headscale 才能生效。

```Bash
sudo systemctl restart headscale
```

顺便看一眼状态，确保它是绿色的（active）：

```Bash
sudo systemctl status headscale
```

---

### 第三步：客户端连接（Tailscale）

现在你的服务端已经准备好了，客户端需要指定 `--login-server` 参数来连接。

#### 1. Linux 客户端（包括你的 ThinkPad 或服务器）

如果你之前连过官方的 Tailscale，建议先重置一下状态：

Bash

```
# 1. 登出并重置
sudo tailscale logout
sudo systemctl stop tailscaled
sudo rm -rf /var/lib/tailscale/tailscaled.state
sudo systemctl start tailscaled

# 2. 连接到你的 Headscale
sudo tailscale up --login-server https://headscale.williamlao.me
```

执行后，终端会给你一个 URL。

注意： 不要直接复制这个 URL 去浏览器！因为它还是会尝试去 Headscale 的机器上找浏览器。

正确的做法：

复制那个命令显示的 nodekey:xxxxx 后面的一长串 URL，或者直接在服务端运行注册命令（见第四步）。

#### 2. Windows 客户端

1. 按 `Win + R`，输入 `cmd`，按回车。
2. 输入以下命令（假设你安装在默认路径）：

    ```bashl
    tailscale logout
    tailscale up --login-server=https://headscale.williamlao.me
    ```
    
3. 它会弹出一个浏览器窗口让你登录，或者给你一个 URL。

#### 3. macOS 客户端 (App Store 版本)

macOS 比较特殊，如果不适用命令行版本：

1. 按住 `Option` (Alt) 键。
2. 点击菜单栏右上角的 Tailscale 图标。
3. 你会发现多了一个 **Debug** 菜单（只有按住 Option 才会出现）。
4. 选择 **Login to Custom Server...**
5. 输入 `https://headscale.williamlao.me`。

#### 4. iOS / Android

1. 打开 App。
2. 狂点右上角的用户头像（或者 Settings）。
3. 找到 **"Accounts"** -> **"Log in to..."**。
4. 点右上角的三个点（Android）或者寻找 **"Use Alternate Server"**。
5. 输入 `https://headscale.williamlao.me`。

---

### 第四步：批准机器（服务端操作）

Headscale 和官方版最大的不同是：**你需要手动批准机器，或者生成 AuthKey。**

当你在客户端运行 `tailscale up ...` 后，它会卡住等待。此时回到你的 **Nginx 所在的那台服务器**：

**方法 A：直接通过节点列表批准（推荐新手）**

```Bash
# 1. 查看有哪些机器在排队
sudo headscale nodes list

# 你会看到一个 ID，比如 1
# 2. 注册这台机器到某个用户（假设用户叫 admin）
# 如果还没创建用户： sudo headscale users create admin
sudo headscale nodes register --user admin --key nodekey:xxxxxx...
```

_(注意：较新版本的 Headscale 命令可能是 `headscale nodes register`，旧版本可能是 `headscale nodes register` 或者直接用 key 注册，具体看 `nodes list` 给出的提示)_

方法 B：预先生成 Auth Key（推荐老手，不用等待）

这是最爽的方法。在服务器上生成一个 key：

```Bash
# 生成一个有效期 24小时的 key
sudo headscale preauthkeys create --user admin --reusable --expiration 24h
```

它会给你一串像 `abcde12345...` 的码。

然后在客户端连接时直接带上：

```Bash
sudo tailscale up --login-server https://headscale.williamlao.me --authkey abcde12345...
```

这样瞬间就连上了，不需要服务器再去批准。

---

### 总结

1. **改配置**：`config.yaml` 里的 `server_url` 必须改成 HTTPS 域名。
2. **重启**：`systemctl restart headscale`。
3. **连接**：客户端加上 `--login-server https://headscale.williamlao.me`。
4. **批准**：服务端 `headscale nodes register`。

# nginx 改为别的端口

没问题。修改对外服务的端口非常简单，但你要同时修改 **Nginx（入口）**、**Headscale（业务声明）** 和 **客户端（连接地址）** 这三个地方，缺一不可。

假设我们要把端口从默认的 **443** 改为 **8443**（这是一个常用的自定义 HTTPS 端口）。

请按以下步骤操作：

### 第一步：修改 Nginx 配置

我们要修改刚才新建的 `/etc/nginx/sites-available/headscale.conf`。

```Bash
sudo nano /etc/nginx/sites-available/headscale.conf
```

你需要改动两处：

1. **SSL 监听端口**：把 `443` 改成 `8443`。
2. **HTTP 跳转逻辑**：如果你还保留 80 端口做跳转，必须把跳转目标加上端口号，否则它默认还是会跳回 443。

**修改后的配置如下（关注注释部分）：**

```Nginx
server {
    server_name headscale.williamlao.me;

    # 【改动 1】监听 8443 端口
    listen 8443 ssl; 
    listen [::]:8443 ssl ipv6only=on; 
    
    # 证书路径保持不变（证书本身不绑定端口）
    ssl_certificate /etc/letsencrypt/live/headscale.williamlao.me/fullchain.pem; 
    ssl_certificate_key /etc/letsencrypt/live/headscale.williamlao.me/privkey.pem; 
    include /etc/letsencrypt/options-ssl-nginx.conf; 
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; 

    location / {
        # 后端不用动，还是指向 Headscale 的 8080
        proxy_pass http://38.57.234.225:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_buffering off;
    }
}

server {
    if ($host = headscale.williamlao.me) {
        # 【改动 2】跳转时显式加上端口号 :8443
        return 301 https://$host:8443$request_uri;
    }

    listen 80;
    listen [::]:80;
    server_name headscale.williamlao.me;
    return 404; 
}
```

保存退出，并重载 Nginx：

```Bash
sudo nginx -t
sudo nginx -s reload
```

---

### 第二步：放行防火墙（非常重要！）

很多时候改了配置连不上，是因为防火墙没开。

如果你用的是 `ufw` (Ubuntu 默认)：

```Bash
sudo ufw allow 8443/tcp
```

如果你使用的是云服务器（阿里云、腾讯云、AWS 等），**必须去控制台的安全组（Security Group）里添加入站规则**，放行 TCP 8443 端口。

---

### 第三步：修改 Headscale 配置

Headscale 必须知道自己的新地址，否则生成的注册链接会是错的。

```Bash
sudo nano /etc/headscale/config.yaml
```

修改 `server_url`：
sj

```YAML
# 加上端口号
server_url: https://headscale.williamlao.me:8443
```

重启 Headscale：

```Bash
sudo systemctl restart headscale
```

---

### 第四步：客户端重新连接

因为服务端的“门牌号”变了，客户端必须更新连接地址。

**服务端（Linux）：**

```Bash
# 只要重新指定 login-server 即可，不需要 logout
sudo tailscale up --login-server https://headscale.williamlao.me:8443
```

其他客户端（Win/Mac/手机）：

你需要重新登出，然后在登录时填写新的 URL：https://headscale.williamlao.me:8443。

---

### 常见疑问

Q: 端口改了，SSL 证书需要重新申请吗？

A: 不需要。 SSL 证书绑定的是域名（williamlao.me），不绑定端口。同一个证书可以用在 443、8443 甚至 9999 端口上，完全通用。

Q: Certbot 自动续签会挂吗？

A: 只要你保留了 80 端口的监听（上面配置代码的第二部分），就不会挂。 Certbot 验证时会访问 `http://你的域名/.well-known/...`。只要 80 端口能通，或者能跳转（Certbot 会跟随跳转），续签就没问题。
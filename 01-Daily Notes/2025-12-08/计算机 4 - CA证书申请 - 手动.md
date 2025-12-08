非常符合你的风格！**“一键化”虽然方便，但它屏蔽了所有的细节，让你不知道发生了什么。** 如果出了问题（比如 Nginx 启动失败），你甚至不知道去哪里修。

要想彻底掌握 HTTPS，我们必须把“申请”和“配置”这两个步骤拆开。我们将采用 **`certonly` (只申请证书)** 模式，然后**手写 Nginx 配置**。

这个过程就像：以前是找中介全包（买房 + 装修），现在我们是**自己去房产局拿房产证**（申请证书），然后**自己买家具装修**（配置 Nginx）。

---

### 第一步：只拿证，不装修 (`certonly` 模式)

我们使用 Certbot 的 `--manual` 模式。这个模式最“硬核”，因为它会把自动化流程暂停，强迫你手动完成“验证挑战”。这能让你看清 ACME 协议到底干了什么。

#### 1. 发起申请

在终端输入以下命令（替换为你自己的域名）：

Bash

```
sudo certbot certonly --manual --preferred-challenges http -d yourdomain.com
```

- `certonly`: 告诉 Certbot，“只给我证书文件，别碰我的 Nginx 配置文件”。
- `--manual`: “我要手动验证”。
- `--preferred-challenges http`: “我要用文件验证的方式（HTTP-01），不要用 DNS 验证”。

#### 2. 面对“挑战” (The Challenge)

命令运行后，Certbot 会初始化，然后它会**暂停**，并在屏幕上打印出一段类似这样的话：

> Create a file containing just this data:
> 
> xR4t... (一长串随机字符)
> 
> And make it available on your web server at this URL:
> 
> http://yourdomain.com/.well-known/acme-challenge/AbCd...

**这就是核心原理：** Let's Encrypt 出了一道题，它说：“如果你真的控制这个网站，你就把这段特定的暗号，放在这个特定的 URL 路径下。我一会去检查。”

#### 3. 手动完成验证

此时**不要按回车**。你需要打开另一个 l 终端窗口，去你的 Nginx 网站根目录（比如 `/var/www/html`）创建这个文件。

```Bash
# 1. 创建目录结构
sudo mkdir -p /var/www/html/.well-known/acme-challenge

# 2. 写入暗号文件 (文件名和内容要复制终端里给你的)
echo "xR4t..." | sudo tee /var/www/html/.well-known/acme-challenge/AbCd...
```

做完后，你可以在浏览器里访问一下那个 URL，看能不能看到那串字符。如果能看到，回到 Certbot 的终端，**按下回车**。

Certbot 会告诉 Let's Encrypt：“好了，去查吧。”

Let's Encrypt 验证通过后，会把证书发给你。

---

### 第二步：认识你的“房产证” (证书文件)

申请成功后，Certbot 会提示你证书保存在了 `/etc/letsencrypt/live/yourdomain.com/` 目录下。

你去 ls 一下，会看到 4 个文件。理解它们是配置 Nginx 的前提：

|**文件名**|**对应概念**|**作用**|**Nginx 配置用哪个？**|
|---|---|---|---|
|**`privkey.pem`**|**私钥 (Private Key)**|绝对机密！服务器用来解密数据的钥匙。|**用！** (`ssl_certificate_key`)|
|**`fullchain.pem`**|**完整证书链**|你的公钥证书 + 中间人证书。发给浏览器看的。|**用！** (`ssl_certificate`)|
|`cert.pem`|仅你的公钥证书|不包含中间人信息，现在的浏览器可能不认。|一般不用|
|`chain.pem`|仅中间人证书|用于构建信任链。|一般不用|

---

### 第三步：手动装修 (配置 Nginx)

现在手里有密钥了，我们要去修改 Nginx 配置，开启 HTTPS 端口（443）。

打开你的配置文件（通常在 `/etc/nginx/sites-available/default` 或 `yourdomain.conf`）。

你需要把原来的配置改成两个部分：

#### 1. 配置 HTTPS (443 端口)

这是主战场。我们需要告诉 Nginx：监听 443，开启 SSL，并指向刚才那两个文件。

```Nginx
server {
    # 1. 监听 443 端口，并启用 SSL
    listen 443 ssl;
    server_name yourdomain.com; # 你的域名

    # 2. 指向证书文件 (这就是你刚才申请到的)
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # 3. SSL 优化配置 (推荐加上，为了安全和性能)
    ssl_protocols TLSv1.2 TLSv1.3; # 只允许安全的协议
    ssl_ciphers HIGH:!aNULL:!MD5;  # 加密套件选择

    # 4. 你的业务逻辑 (反向代理)
    location / {
        proxy_pass http://127.0.0.1:8000;
        
        # 别忘了传递真实 IP，否则后端只能看到 127.0.0.1
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

#### 2. 配置 HTTP 强转 HTTPS (80 端口)

我们要保证用户输入 `http://` 时自动跳到 `https://`。

```Nginx
server {
    listen 80;
    server_name yourdomain.com;
    
    # 301 重定向：永久跳转
    return 301 https://$host$request_uri;
}
```

---

### 第四步：验收与重启

配置写完了，千万别直接重启，先测试语法。

1. **测试配置**：

    ```Bash
    sudo nginx -t
    ```
    
    如果显示 `syntax is ok` 和 `test is successful`，说明你写对了。
    
2. **重载服务**：

    ```Bash
    sudo nginx -s reload
    ```

此时，你的手动 HTTPS 之旅就完成了！

---

### ⚠️ 一个致命的细节：关于“续期”

刚才我们用了 --manual 模式。这有一个巨大的副作用：

它无法自动续期！

因为 `--manual` 需要你手动创建文件、手动按回车。90 天后证书过期时，`certbot renew` 命令会报错，因为它无法自动完成那个“手动创建文件”的动作。

**生产环境的最佳实践（折中方案）：`--webroot` 模式**

如果你想既拥有“手动配置 Nginx”的掌控感，又想拥有“自动续期”的便利，申请证书时应该用这个命令：

```Bash
sudo certbot certonly --webroot -w /var/www/html -d yourdomain.com
```

- `--webroot`: 告诉 Certbot，“我不暂停了，你直接把那个验证文件自动写到 `/var/www/html` 这个目录下，然后你自己去验证”。
- 前提是：你的 Nginx 80 端口必须已经配置好，并且指向了 `/var/www/html`。

这样，你既理解了原理，手动配置了 Nginx 的 SSL 参数，又保留了 Let's Encrypt 的自动化续期能力。这就是**专业玩家**的做法。

你现在想试试用 `certonly` 走一遍流程吗？还是对 Nginx 的 `proxy_set_header` 那些参数感兴趣？
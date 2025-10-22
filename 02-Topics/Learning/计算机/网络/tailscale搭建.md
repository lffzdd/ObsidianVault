好的，这是一个非常典型且有价值的场景。结合我们之前讨论的所有技术，为您朋友在中国大陆的特定情况提供一个最佳实践方案。

**核心问题分析：**

1. **目标：** 让分布在不同网络下的多台主机（Linux/Ubuntu）可以像在同一个局域网里一样互相通信。
    
2. **资产：** 一台有公网 IP 的 ECS 云主机、一个域名、多台内网主机。
    
3. **限制：** ECS 的公网带宽昂贵，不能作为所有流量的中心转发节点，否则成本会爆炸。
    
4. **环境：** 中国大陆。这意味着国际网络连接延迟高且可能不稳定，应优先考虑纯国内方案。
    

---

### 最佳方案：自建 Headscale + DERP 服务器

对于您朋友的这个场景，我**强烈不推荐**使用 Tailscale 官方服务或 Zerotier 的官方服务。因为它们的控制服务器和默认中继服务器都在海外，会导致：

- **高延迟：** 客户端与控制服务器的信令交互慢，网络状态更新不及时。
    
- **不确定性：** 跨国网络连接可能受到干扰，影响稳定性。
    
- **低效中继：** 如果设备间无法直连，流量需要绕道海外的 DERP 服务器再回来，路径长、速度慢。
    

因此，最理想、最经济、最稳定的方案是：**利用他已有的 ECS 和域名，自建一套完全属于自己的 Tailscale 网络。**

这套方案完美地解决了他的痛点：

- **经济省钱：** ECS 只承载极少量信令交互（控制平面）和偶尔的应急中-继流量，绝大部分数据将通过设备间的 P2P 直连传输，几乎不消耗 ECS 带宽。
    
- **高效稳定：** 所有核心服务都在国内，网络延迟极低，连接稳定可靠。
    
- **安全可控：** 所有网络元数据都掌握在自己手中。
    

---

### 实施步骤详解

以下是详细的操作步骤和指南。

#### **准备工作 (Prerequisites)**

1. **ECS 服务器：**
    
    - **配置要求：** Headscale 和 DERP 服务本身资源消耗极低，一台基础的 **1核 CPU / 1GB 内存** 的 ECS 就完全足够了。
        
    - **操作系统：** 推荐 Ubuntu 22.04 LTS 或 Debian 11/12。
        
    - **安全组/防火墙：** 确保开放以下端口：
        
        - `TCP 80`: 用于申请 SSL 证书。
            
        - `TCP 443`: 用于 Headscale 的 HTTPS API 和 DERP 的 HTTPS 服务。
            
        - `UDP 3478`: 用于 DERP 的 STUN 服务。
            
        - `UDP 49152-65535` (一个范围): 也可以只开一个特定端口，如 `UDP 54321`，用于 DERP 的 TURN over UDP 服务。
            
2. **域名：**
    
    - 准备一个域名，并创建两个 A 记录，都指向 ECS 的公网 IP 地址。例如：
        
        - `headscale.yourdomain.com` -> `ECS 公网 IP`
            
        - `derp.yourdomain.com` -> `ECS 公网 IP`
            

#### **第一步：在 ECS 上部署 Headscale**

最简单的方式是使用 Docker 和 Docker Compose。

1. **安装 Docker 和 Docker Compose:**
    
    Bash
    
    ```
    # 安装 Docker
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    # 安装 Docker Compose
    sudo apt-get update && sudo apt-get install docker-compose-plugin
    ```
    
2. **配置 Headscale:**
    
    - 创建一个目录，例如 `mkdir ~/headscale && cd ~/headscale`。
        
    - 在该目录下创建一个 `docker-compose.yml` 文件。
        
    - 在同一目录下创建一个 `config` 文件夹，并在其中创建一个 `config.yaml` 文件。
        
    
    **`config.yaml` 核心配置示例：**
    
    YAML
    
    ```
    server_url: https://headscale.yourdomain.com
    listen_addr: 0.0.0.0:8080 # 在容器内监听8080
    metrics_listen_addr: 0.0.0.0:9090
    
    # 使用Let's Encrypt自动获取HTTPS证书
    tls_letsencrypt_hostname: headscale.yourdomain.com
    tls_letsencrypt_cache_dir: /var/lib/headscale/cache
    tls_letsencrypt_challenge_type: HTTP-01
    
    # 数据库配置（使用内嵌的SQLite，简单方便）
    db_type: sqlite3
    db_path: /var/lib/headscale/db.sqlite
    
    # IP地址段配置
    ip_prefixes:
      - 100.64.0.0/10
    
    # 启用 OIDC (可选，更高级的登录方式)
    # oidc: ...
    ```
    
    **`docker-compose.yml` 示例：**
    
    YAML
    
    ```
    version: '3.8'
    services:
      headscale:
        image: headscale/headscale:latest
        container_name: headscale
        volumes:
          - ./config:/etc/headscale
          - ./data:/var/lib/headscale
        ports:
          - "80:8080"  # 临时用于Let's Encrypt HTTP验证
          - "443:443"  # 如果使用反向代理
          # 如果直接暴露，需要根据config.yaml来调整
        command: headscale serve
        restart: unless-stopped
    ```
    
    _注意：使用 Let's Encrypt 直接由 Headscale 签发证书是最简单的方式之一，但生产环境更推荐使用 Nginx 或 Caddy 作为反向代理来处理 TLS。_
    
3. **启动 Headscale:**
    
    Bash
    
    ```
    docker compose up -d
    ```
    

#### **第二步：(强烈推荐) 部署 DERP 中继服务器**

在**同一台 ECS** 上，我们可以再部署一个 DERP 服务器，这样当设备间 P2P 连接失败时，有一个国内的、低延迟的中继兜底。

1. **获取 `derper`:** Tailscale 官方提供了 derper 的二进制文件或 Docker 镜像。同样，使用 Docker Compose 最为方便。
    
2. **修改 `docker-compose.yml` 添加 derper 服务：**
    
    YAML
    
    ```
    # ... headscale 服务部分 ...
      derper:
        image: ghcr.io/tailscale/derper
        container_name: derper
        hostname: derp.yourdomain.com
        ports:
          - "3478:3478/udp" # STUN
          - "443:443" # 如果不和headscale冲突，需要用Nginx/Caddy分流
        # 如果443端口冲突，可以映射到其他端口，如 "8443:443"
        volumes:
          - ./derper/certs:/app/certs
        environment:
          - DERP_DOMAIN=derp.yourdomain.com
          # - DERP_CERT_MODE=letsencrypt # 可以自动获取证书
          # - DERP_ADDR=:443
        restart: unless-stopped
    ```
    
    _通常，将 Headscale 和 DERP 都放在同一个 Nginx 或 Caddy 反向代理后面，通过域名来区分流量，是最佳实践。_
    
3. 让 Headscale 知道私有 DERP 的存在：
    
    在 Headscale 的 config.yaml 中添加 derp 部分：
    
    YAML
    
    ```
    derp:
      server:
        enabled: true
        region_id: 999
        region_code: "ecs-private"
        region_name: "My Private ECS DERP"
      urls:
        - https://derp.yourdomain.com
      stun_port: 3478
    ```
    
    重启 Headscale 服务使其生效。
    

#### **第三步：连接所有主机**

现在，在他每一台 Linux/Ubuntu 主机上：

1. **安装 Tailscale 客户端：**
    
    Bash
    
    ```
    curl -fsSL https://tailscale.com/install.sh | sh
    ```
    
2. **登录到自建的 Headscale 服务器：**
    
    Bash
    
    ```
    # 替换成你自己的Headscale域名
    sudo tailscale up --login-server=https://headscale.yourdomain.com
    ```
    
3. **在 ECS 服务器上授权新主机：**
    
    - 执行 `docker exec -it headscale headscale nodes list` 查看待授权的主机。
        
    - 执行 `docker exec -it headscale headscale nodes register --key [node_key]` 来授权。更简单的方式是创建用户并注册。
        
    
    Bash
    
    ```
    # 创建一个用户/命名空间
    docker exec -it headscale headscale users create myfriend
    # 列出节点并注册
    docker exec -it headscale headscale nodes list
    docker exec -it headscale headscale nodes register --user myfriend --key [node_key]
    ```
    

#### **第四步：验证和使用**

在任何一台主机上运行 `tailscale status`，他应该能看到所有已连接的主机及其 `100.x.x.x` 格式的 IP 地址。

运行 `tailscale ping [另一台主机的名称或IP]`，可以看到连接类型是 `direct`（直连）还是 `relay`（通过你的 DERP 中继）。

现在，他就可以直接使用这些 `100.x.x.x` 的 IP 地址在所有主机间进行任何网络操作了，例如：

- `ssh user@100.x.y.z`
    
- `scp file.txt user@100.x.y.z:/path/`
    
- 在一台主机上运行数据库，在另一台主机上连接它。
    

这个方案将前期的一次性配置工作，转化为了长期的、几乎零成本的、高效稳定的内网通信体验。
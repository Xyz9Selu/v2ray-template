# V2Ray 模板

快速搭建带有网页前端的 V2Ray/XRay 代理服务的模板。

## 项目结构

```
.
├── client/                 # 客户端配置
│   ├── config.json        # 客户端配置文件（从模板生成）
│   └── docker-compose.yaml # 客户端 docker compose 文件
├── server/                # 服务端配置
│   ├── config.json        # 服务端配置文件（从模板生成）
│   └── docker-compose.yaml # 服务端 docker compose 文件
├── web/                   # 网页前端（DokuWiki）
│   └── docker-compose.yaml # Web 服务 docker compose 文件
└── nginx/                 # Nginx 配置
    └── sites-enabled/     # Nginx 站点配置
        ├── dokuwiki      # DokuWiki 站点配置
        └── v2ray         # V2Ray 代理配置
```

## 安装说明

### 服务端设置

1. 复制 `server/config.json.template` 到 `server/config.json`
2. 更新配置文件中的设置：
   - 将 `<your-user-id>` 替换为你的 UUID
   - 将 `<your-vless-server-host>` 替换为你的域名
   - 将 `<your-vless-server-path>` 替换为你的 WebSocket 路径

3. 启动服务端：
```bash
cd server
docker-compose up -d
```

### 客户端设置

1. 复制 `client/config.json.template` 到 `client/config.json`
2. 使用与服务端相同的设置更新配置：
   - 将 `<your-user-id>` 替换为你的 UUID
   - 将 `<your-vless-server-host>` 替换为你的域名
   - 将 `<your-vless-server-path>` 替换为你的 WebSocket 路径

3. 启动客户端：
```bash
cd client
docker-compose up -d
```

### Web 前端设置

1. 更新 `web/docker-compose.yaml` 中的环境变量：
   - DOKUWIKI_USERNAME（用户名）
   - DOKUWIKI_PASSWORD（密码）
   - DOKUWIKI_FULL_NAME（全名）
   - DOKUWIKI_WIKI_NAME（Wiki 名称）
   - DOKUWIKI_EMAIL（电子邮箱）

2. 启动 Web 服务：
```bash
cd web
docker-compose up -d
```

### Nginx 设置

1. 如果未安装 Nginx，先安装：
```bash
sudo apt update
sudo apt install nginx
```

2. 复制 Nginx 配置文件：
```bash
sudo cp nginx/sites-enabled/v2ray /etc/nginx/sites-enabled/
```

3. 更新配置文件中的域名：
   - 在 `nginx/sites-enabled/v2ray` 中：
     - 将 `<your-vless-server-host>` 替换为你的域名
     - 将 `<your-vless-server-path>` 替换为你的 WebSocket 路径
     - 默认配置包含：
       - 80 端口用于 HTTP
       - 13080 端口反向代理到 DokuWiki
       - 33685 端口用于 V2Ray 的 WebSocket 代理

4. 测试并重新加载 Nginx：
```bash
sudo nginx -t
sudo systemctl reload nginx
```

### SSL 证书设置（Certbot）

1. 安装 Certbot 和 Nginx 插件：
```bash
sudo apt install certbot python3-certbot-nginx
```

2. 获取 SSL 证书：
```bash
sudo certbot --nginx -d your-domain.com
```

3. Certbot 将自动：
   - 获取 SSL 证书
   - 更新 Nginx 配置
   - 设置自动续期

4. 验证自动续期：
```bash
sudo systemctl status certbot.timer
sudo certbot renew --dry-run
```

### VLESS 链接格式

服务器设置完成后，你可以生成 VLESS 链接供客户端导入：

```
vless://<user-id>@<domain>:443?encryption=none&security=tls&type=ws&host=<domain>&path=/<ws-path>#<remarks>
```

示例：
```
vless://41c5b71b-4797-425b-8c27-fc1d47453c27@example.com:443?encryption=none&security=tls&type=ws&host=example.com&path=/websocket#MyProxy
```

参数说明：
- `<user-id>`：配置文件中的 UUID
- `<domain>`：你的服务器域名
- `<ws-path>`：配置文件中的 WebSocket 路径
- `<remarks>`：可选的备注信息

## 注意事项

- 所有服务都配置为自动重启（除非手动停止）
- 确保配置文件的安全性，切勿将其提交到版本控制系统
- Web 前端是可选的，如不需要可以禁用
- SSL 证书在 Certbot 正确配置的情况下会自动续期
- 记得更新防火墙设置以允许 HTTPS 流量（443 端口）

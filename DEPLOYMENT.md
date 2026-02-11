# 部署文档

## 📦 部署选项

### 1. 本地部署

最简单的方式，无需任何配置：

```bash
# 克隆仓库
git clone https://github.com/mason0510/ball-puzzle-game.git
cd ball-puzzle-game

# 直接用浏览器打开
open index.html  # macOS
# 或
start index.html  # Windows
# 或双击 index.html 文件
```

### 2. 使用 Python HTTP 服务器

如果需要本地 HTTP 服务器（避免 CORS 问题）：

```bash
# Python 3
python3 -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000

# 访问
open http://localhost:8000
```

### 3. 使用 Node.js http-server

```bash
# 全局安装
npm install -g http-server

# 运行
http-server -p 8000

# 访问
open http://localhost:8000
```

### 4. 部署到静态托管平台

#### GitHub Pages

1. 推送代码到 GitHub 仓库
2. 进入仓库 Settings → Pages
3. 选择 main 分支
4. 点击 Save
5. 访问 `https://yourusername.github.io/ball-puzzle-game/`

#### Netlify

```bash
# 安装 Netlify CLI
npm install -g netlify-cli

# 登录
netlify login

# 部署
netlify deploy --prod

# 或者直接拖拽文件夹到 Netlify 网站
```

#### Vercel

```bash
# 安装 Vercel CLI
npm install -g vercel

# 登录
vercel login

# 部署
vercel --prod
```

#### Cloudflare Pages

1. 登录 Cloudflare Dashboard
2. 进入 Pages
3. 创建新项目
4. 连接 GitHub 仓库
5. 构建命令留空（静态文件）
6. 输出目录: `/`
7. 点击部署

### 5. 部署到 Linux 服务器

#### 使用 Nginx

```bash
# 安装 Nginx
sudo apt update
sudo apt install nginx

# 复制文件
sudo cp index.html /var/www/html/ball-puzzle.html

# 或配置独立站点
sudo mkdir -p /var/www/ball-puzzle
sudo cp index.html /var/www/ball-puzzle/

# 配置 Nginx
sudo nano /etc/nginx/sites-available/ball-puzzle

# 添加配置
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/ball-puzzle;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# 启用站点
sudo ln -s /etc/nginx/sites-available/ball-puzzle /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

#### 使用 Caddy

```bash
# 安装 Caddy
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy

# 复制文件
sudo mkdir -p /var/www/ball-puzzle
sudo cp index.html /var/www/ball-puzzle/

# 配置 Caddyfile
sudo nano /etc/caddy/Caddyfile

# 添加配置
your-domain.com {
    root * /var/www/ball-puzzle
    file_server
}

# 重启 Caddy
sudo systemctl reload caddy
```

#### 使用 Docker

```bash
# 创建 Dockerfile
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# 构建镜像
docker build -t ball-puzzle-game .

# 运行容器
docker run -d -p 8080:80 ball-puzzle-game

# 访问
open http://localhost:8080
```

## 🌐 生产环境部署实例

### 腾讯云服务器部署（当前生产环境）

**服务器信息**：
- IP: 111.229.63.121
- 系统: Ubuntu 22.04
- Web服务器: Caddy

**部署步骤**：

```bash
# SSH 连接
ssh ubuntu@111.229.63.121

# 创建目录
sudo mkdir -p /var/www/files

# 上传文件（本地执行）
scp index.html ubuntu@111.229.63.121:/tmp/ball-puzzle.html
ssh ubuntu@111.229.63.121 "sudo mv /tmp/ball-puzzle.html /var/www/files/"

# Caddy 配置已存在（端口 80）
# 配置文件: /etc/caddy/Caddyfile
# 重载 Caddy
ssh ubuntu@111.229.63.121 "sudo systemctl reload caddy"
```

**访问地址**：
- http://111.229.63.121/ball-puzzle.html

**Caddy 配置片段**：
```caddy
http://111.229.63.121 {
    root * /var/www/files
    file_server browse
}
```

## 🔧 故障排查

### 游戏无法加载

1. 检查浏览器控制台错误
2. 确认文件路径正确
3. 检查服务器配置（如果使用服务器）

### 音乐/音效无声

1. 检查浏览器是否允许自动播放音频
2. 点击页面任意位置激活音频上下文
3. 检查音乐开关是否关闭

### 服务器部署 502/404 错误

1. 确认文件已正确上传
2. 检查文件权限: `sudo chmod 644 /var/www/files/ball-puzzle.html`
3. 检查 Web 服务器配置
4. 查看服务器错误日志

### 端口被防火墙阻断

```bash
# 检查端口开放状态
sudo netstat -tlnp | grep :80

# 检查防火墙规则（UFW）
sudo ufw status

# 检查 iptables
sudo iptables -L -n

# 云服务器需在控制台安全组开放端口
# 腾讯云: 安全组 → 入站规则 → 添加规则
# 阿里云: 安全组 → 配置规则 → 添加安全组规则
```

## 📊 性能优化建议

### 1. 启用 Gzip 压缩

**Nginx**:
```nginx
gzip on;
gzip_types text/html application/javascript text/css;
```

**Caddy**:
```caddy
encode gzip
```

### 2. 设置缓存头

**Nginx**:
```nginx
location ~* \.(html|js|css)$ {
    expires 1d;
    add_header Cache-Control "public, immutable";
}
```

**Caddy**:
```caddy
header /index.html Cache-Control "max-age=86400"
```

### 3. 使用 CDN

- Cloudflare CDN（免费）
- AWS CloudFront
- 阿里云 CDN
- 腾讯云 CDN

## 🔒 安全建议

1. **使用 HTTPS**（生产环境必须）
2. **设置安全头**：
   ```
   X-Frame-Options: DENY
   X-Content-Type-Options: nosniff
   X-XSS-Protection: 1; mode=block
   ```
3. **限制文件访问**（仅允许 HTML 文件）
4. **定期更新服务器和依赖**

## 📝 更新部署

```bash
# 本地更新代码
git pull origin main

# 重新部署到服务器
scp index.html ubuntu@111.229.63.121:/tmp/ball-puzzle.html
ssh ubuntu@111.229.63.121 "sudo mv /tmp/ball-puzzle.html /var/www/files/ && sudo systemctl reload caddy"

# 或使用自动化脚本（见下方）
```

## 🤖 自动化部署脚本

创建 `deploy.sh`:

```bash
#!/bin/bash
set -e

SERVER="ubuntu@111.229.63.121"
REMOTE_PATH="/var/www/files/ball-puzzle.html"

echo "📦 Uploading game file..."
scp index.html $SERVER:/tmp/ball-puzzle.html

echo "🚀 Deploying to production..."
ssh $SERVER "sudo mv /tmp/ball-puzzle.html $REMOTE_PATH && sudo systemctl reload caddy"

echo "✅ Deployment complete!"
echo "🌐 Visit: http://111.229.63.121/ball-puzzle.html"
```

使用方式：
```bash
chmod +x deploy.sh
./deploy.sh
```

---

如有部署问题，欢迎提 [Issue](https://github.com/mason0510/ball-puzzle-game/issues)！

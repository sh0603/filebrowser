基于 Docker 的 FileBrowser 详细部署指南
一、项目结构准备
创建filebrowser目录
mkdir -p ~/filebrowser/{config,data}
cd ~/filebrowser

二、Dockerfile方法
1. 创建 `Dockerfile`

# ~/filebrowser/Dockerfile
FROM filebrowser/filebrowser:latest

# 设置默认配置和环境变量
ENV FB_PORT=80 \
    FB_ROOT=/srv \
    FB_DATABASE=/config/filebrowser.db

# 创建非 root 用户并设置权限
RUN addgroup -g 1000 filebrowser && \
    adduser -u 1000 -G filebrowser -D filebrowser

# 切换用户
USER filebrowser

# 数据卷声明
VOLUME ["/config", "/srv"]

# 暴露端口
EXPOSE 80

# 启动命令
CMD ["filebrowser", "--config", "/config/filebrowser.json"]

2. 构建镜像

docker build -t my-filebrowser .

3. 启动容器
```bash
docker run -d \
  --name filebrowser \
  -p 8082:80 \
  -v ~/filebrowser/config:/config \
  -v /home:/srv \
  -e FB_BASEURL=/files \
  my-filebrowser
三、编写docker-compose
1.创建 `docker-compose.yml`

# ~/filebrowser/docker-compose.yml
version: '3.8'

services:
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: my-filebrowser
    restart: unless-stopped
    user: "1000:1000"  # 指定非 root 用户
    environment:
      - FB_DATABASE=/config/filebrowser.db
      - FB_BASEURL=/files  # 自定义 URL 路径
    ports:
      - "8082:80"
    volumes:
      - ./config:/config    # 配置文件目录
      - /home:/srv          # 托管文件目录
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

2. 设置目录权限
# 确保宿主机目录权限正确
chown -R 1000:1000 ~/filebrowser/config
chmod -R 755 /home

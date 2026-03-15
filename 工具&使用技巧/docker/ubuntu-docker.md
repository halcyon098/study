官方推荐的安装方法。
在 Ubuntu 上安装 Docker Desktop 的推荐方法：
## 搭建 Docker 的包仓库。 
参见第一步 用 apt 仓库安装 。
1.搭建 Docker 的 apt 仓库。
```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```
2.安装 Docker 包
最新版本
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
其他版本
要安装特定版本的 Docker Engine，首先列出仓库中的可用版本：
```bash
apt list --all-versions docker-ce

docker-ce/noble 5:29.2.1-1~ubuntu.24.04~noble <arch>
docker-ce/noble 5:29.2.0-1~ubuntu.24.04~noble <arch>
...
```
选择所需版本并安装：
```bash
VERSION_STRING=5:29.2.1-1~ubuntu.24.04~noble
sudo apt install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```
Docker 服务在安装后会自动启动。要验证 Docker 是否在运行，请使用：
```bash
sudo systemctl status docker
```
有些系统可能关闭了该行为，需要手动启动：
```bash
sudo systemctl start docker
```

3.通过运行 hello-world 镜像验证安装成功：
```bash
sudo docker run hello-world
```

## 下载最新的 DEB 软件包 
https://desktop.docker.com/linux/main/amd64/docker-desktop-amd64.deb?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-linux-amd64&_gl=1*txd5yg*_gcl_au*OTQ0NjA4ODE2LjE3NzA2MDUyNDU.*_ga*MjExMzA3OTg5NS4xNzcwNjA1MjQ1*_ga_XJWPQMJYHQ*czE3NzA4NzQ4OTckbzIkZzEkdDE3NzA4NzQ5OTIkajQ5JGwwJGgw
## 用 apt 安装该软件包
```bash
sudo apt-get update
sudo apt install ./docker-desktop-amd64.deb
```

安装过程结束时，apt 显示错误，原因是安装了下载的软件包。你可以忽略这个错误信息。
```bash
N: Download is performed unsandboxed as root, as file '/home/user/Downloads/docker-desktop.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
```
默认情况下，Docker Desktop 安装在 /opt/docker-desktop。

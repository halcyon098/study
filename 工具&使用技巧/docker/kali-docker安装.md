下面给你一份 **在 Kali Linux 上从零配置 Docker 环境的可复制安装指南**，包含需要的包、推荐的安装来源（官方仓库优先）、安装命令、常见后处理（非 root 使用、启动/验证）、以及常见问题快速排查。直接复制粘贴下面的命令到 Kali 终端即可执行（按顺序）。

> 说明：Kali 基于 Debian/Ubuntu，推荐使用 Docker 官方仓库安装 `docker-ce` 系列包（比 distro 自带的 `docker.io` 更新、更完整）。如果你想用更简单的单包 `docker.io`，我也会给出替代命令。

---

# 安装前准备（移除旧包 & 安装依赖）

先清理旧版本并安装必需工具：

```bash
sudo apt update
sudo apt remove -y docker docker-engine docker.io containerd runc  # 移除旧残留（若存在）
sudo apt install -y ca-certificates curl gnupg lsb-release apt-transport-https
```

---

# 选择 A：使用 Docker 官方仓库（推荐）

1. **添加 Docker 官方 GPG Key 与仓库**
    

```bash
# 添加 GPG key
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加 apt repository（Kali 基于 Debian；用 lsb_release 获取 codename）
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/debian $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

2. **更新 apt 并安装推荐包（Engine + CLI + containerd + Compose & Buildx 插件）**
    

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

3. **启用并启动 docker 服务**
    

```bash
sudo systemctl enable --now docker
# 检查状态
sudo systemctl status docker --no-pager
```

---

# 选择 B：使用 Kali/Ubuntu 自带包（更简单但可能版本旧）

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
# 或仅 docker.io 并用 docker compose plugin：
# sudo apt install -y docker.io
```

> 注意：`docker-compose`（带连字符）通常是旧版 Python 实现；推荐使用官方插件 `docker-compose-plugin`（命令 `docker compose`）。

---

# 安装后常用配置（让普通用户运行 docker）

```bash
# 把当前用户加入 docker 组（登出/重启会话后生效）
sudo usermod -aG docker $USER

# 立即生效（当前 shell 临时）
newgrp docker
```

---

# 验证安装

```bash
# 查看 docker 版本
docker version

# 运行测试容器
docker run --rm hello-world

# 验证 compose 插件
docker compose version
# 或旧命令（若安装了旧版）
docker-compose --version
```

---

# 常见需要的包（汇总）

- `docker-ce` —— Docker Engine（dockerd）
    
- `docker-ce-cli` —— docker 命令行
    
- `containerd.io` —— containerd 运行时
    
- `runc` —— OCI runtime（一般随 containerd 提供）
    
- `docker-buildx-plugin` —— multi-platform / buildx
    
- `docker-compose-plugin` —— 新版 Compose (`docker compose`)
    
- `docker.io` —— Debian/Ubuntu 自带的整合包（替代 `docker-ce`，但版本往往较旧）
    
- 辅助：`ca-certificates` `curl` `gnupg` `lsb-release` `apt-transport-https`
    

---

# 可选：启用 rootless 模式（非必须）

如果你想在非 root 下运行 daemon（rootless），可参考：

```bash
# 安装 rootless extras（若官方仓库提供）
sudo apt install -y docker-ce-rootless-extras

# 使用 dockerd-rootless.sh 启动（需参照官方文档步骤）
```

（rootless 需要额外配置用户命名空间、systemd user 服务等；有时在 Kali 上会遇到兼容性问题）

---

# 常见问题快速排查

- **`docker: Got permission denied while trying to connect to the Docker daemon socket`**  
    -> 把用户加入 `docker` 组并重新登录：`sudo usermod -aG docker $USER`，然后重新登录 shell。
    
- **服务无法启动（systemctl status docker 报错）**  
    -> 查看 `journalctl -u docker --no-pager -n 200` 日志，检查 cgroup、overlayfs、containerd 报错。Kali 使用 systemd，确认内核支持 cgroup v1/v2，或调整 containerd 配置以兼容。
    
- **构建多平台时报错**  
    -> 确认已安装 `docker-buildx-plugin`，并确保 QEMU 用户空间静态二进制（`docker run --rm --privileged tonistiigi/binfmt --install all`）已配置（只在需要 cross-build 时用）。
    
- **`docker compose` 命令不可用但 `docker` 可用**  
    -> 安装 `docker-compose-plugin`：`sudo apt install docker-compose-plugin`，使用 `docker compose`（注意没有 “-”）。
    

---

# 安全/生产建议

- 在生产环境请使用 Docker 官方仓库并定期更新（security updates）。
    
- 关闭不必要的远程 API（不要把 `dockerd` 绑定到公网）。
    
- 使用镜像签名 / 私有 registry / 镜像扫描（docker-scan-plugin）提升供应链安全。
    
- 如果在 CI/CD 中运行 docker-in-docker，优先使用 `docker buildx` 或者 DinD 注意隔离风险。
    

---

如果你愿意，我可以：

1. 根据你当前 Kali 的具体版本（`cat /etc/os-release`）生成精确的安装命令并检查兼容性；
    
2. 给出一个带注释的 `setup-docker.sh` 脚本供你一键执行（含错误检测与可选项）；
    
3. 帮你配置 Docker 使用加速镜像或本地私有 registry（如果你在中国大陆或需要加速镜像拉取）。
    

要哪一项就直接告诉我（我马上给出脚本/命令）。
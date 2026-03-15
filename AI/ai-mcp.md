在linux虚拟机上尝试使用mcp配合ai进行代码编写，参考

https://docs.packyapi.com/docs/faq/Codex.html#%E5%9C%A8windows%E7%B3%BB%E7%BB%9F%E4%B8%8B-%E4%B8%9D%E6%BB%91%E4%BD%BF%E7%94%A8codex

我使用的环境是在代码和codex的key都在debian虚拟机上，本机使用vscode远程连接功+vscode的codex插件。

此方法同时解决读写文件、乱码、Token耗费高、项目无记忆等多个痛点

确保你的Codex CLI 与Vscode Codex插件正常运行，即你已经能顺利在Vscode的Codex插件上与模型进行对话

我们需要安装两个MCP工具：

Serena： 功能强大的编码代理工具包，提供语义检索和项目记忆功能 Github

Desktop-Commander： 一个优秀的文件操作工具 Github

```bash
cd ~\.codex
```

codex的config.yaml

```yaml
#.....codex其他基础配置
# desktop-commander：stdio + docker
[mcp_servers.desktop-commander]
type = "stdio"
command = "docker"
startup_timeout_sec = 50
args = [
  "run", "-i", "--rm",
  "-v", "dc-system:/usr",
  "-v", "dc-home:/root",
  "-v", "dc-workspace:/workspace",
  "-v", "dc-packages:/var",
  "-v", "/project:/mnt/kf",
  "mcp/desktop-commander:latest"
]

# Serena
[mcp_servers.Serena]
type = "http"
url = "http://127.0.0.1:9121/mcp"
```

AGENTS.md：

```markdown
# Codex全局工作指南

## 回答风格:
 - 回答必须使用中文
 - 对总结、Plan、Task、以及长内容的输出，优先进行逻辑整理后使用美观的Table格式整齐输出;普通内容正常输出

## 工具使用:
1. 文件与代码检索:使用serena mcp来进行文件与代码的检索
2. 文件相关操作:对文件的创建、读取、编辑、删除等操作
    - 优先使用apply_patch工具进行
    - 读文件，apply_patch工具报错或出现问题的情况下使用desktop-commander mcp
    - 任何情况下，禁止使用cmd、powershell或者python来进行文件相关操作
```

"任何情况下，禁止使用cmd、powershell或者python来进行文件相关操作",这个是windows上的，修改或者直接去掉即可。

## DesktopCommanderMCP

https://github.com/wonderwhy-er/DesktopCommanderMCP

这个我直接在docker中拉取的，比较稳定好用。

```bash
docker pull mcp/desktop-commander:latest
```

直接用ai启动docker即可：

```yaml
# desktop-commander：stdio + docker
[mcp_servers.desktop-commander]
type = "stdio"
command = "docker"
startup_timeout_sec = 50
args = [
  "run", "-i", "--rm",
  "-v", "dc-system:/usr",
  "-v", "dc-home:/root",
  "-v", "dc-workspace:/workspace",
  "-v", "dc-packages:/var",
  "-v", "/project:/mnt/project",
  "mcp/desktop-commander:latest"
]
```

注意"-v", "/project:/mnt/project",这个命令，进行挂载即可。

## Serena

https://github.com/oraios/serena

官方文档：https://oraios.github.io/serena/01-about/000_intro.html

这个开始使用docker环境，太多坑了，填不完，使用本地运行了。

python环境有要求，要<=3.11.x，我使用的3.11.9

```bash
#拉源码，配置运行环境
git clone https://github.com/ai-mcp/serena.git
cd serena
python --version
python3 -m venv venv
source venv/bin/activate
pip install -e .

curl -LsSf https://astral.sh/uv/install.sh | sh #uv/uvx

uv --version
uvx --version

#运行命令
uv run serena start-mcp-server --context codex --transport streamable-http --port 9121

#限定项目目录的话可以使用这个参数
--project /workspace/project
```

这样就启动了一个本地Serena的http服务。

```yaml
# Serena
[mcp_servers.Serena]
type = "http"
url = "http://127.0.0.1:9121/mcp"
```

## codex

使用vscode之前，先用cli在项目目录下启动看一下能否拉起访问到这两个mcp工具。

使用之前给codex一个指令：

```txt
通过 Serena 将当前目录设为项目并激活
```

官方文档的原话：
After codex has started, you need to activate the project, which you can do by saying:

“Activate the current dir as project using serena”

后面基本就可以使用了。


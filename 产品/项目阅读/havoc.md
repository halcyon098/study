# Havoc C 2 框架学习指南

Havoc 是一个现代化的后渗透 C 2 框架，采用 **Teamserver-Client-Agent** 三层架构。以下是建议的学习路径：

---

## 📋 项目架构概览

```
┌─────────────────┐         WebSocket         ┌──────────────────┐
│  Havoc Client   │  ◄────────────────────►   │  Teamserver      │
│   (C++/Qt)      │                           │    (Go)          │
└─────────────────┘                           └────────┬─────────┘
                                                       │ HTTP/SMB
                                                       ▼
                                              ┌──────────────────┐
                                              │   Demon Agent    │
                                              │    (C/ASM)       │
                                              └──────────────────┘
```

**技术栈：**
- **Teamserver**: Go (Cobra CLI, WebSocket, HTTP/SMB handlers)
- **Client**: C++ (Qt 5/6, WebSocket client)
- **Demon Agent**: C + ASM (Windows API, Syscalls, Sleep Obfuscation)

---

## 📚 推荐学习路径

### 第一阶段：整体架构理解 (1-2 天)

**1. 项目入口和配置**
```
建议阅读顺序:
├── README.md                    # 项目整体介绍
├── makefile                     # 构建流程
├── profiles/havoc.yaotl         # C2 配置文件示例
└── payloads/Demon/README.md     # Agent 介绍
```

**重点理解:**
- C 2 Profile 结构 (Listeners, Agents, 通信配置)
- 项目构建流程
- 各组件交互方式

---

### 第二阶段：Teamserver 服务端 (3-5 天)

**2. 服务端入口和命令行**
```
teamserver/
├── main.go                      # 程序入口
├── cmd/
│   ├── cmd.go                   # CLI 命令定义 (Cobra)
│   ├── server.go                # server 子命令
│   └── client.go                # client 子命令 (连接模式)
```

**3. 核心服务模块**
```
teamserver/cmd/server/
├── teamserver.go                # Teamserver 核心初始化 ⭐
├── dispatch.go                  # 消息分发处理 ⭐
├── agent.go                     # Agent 管理
├── listener.go                  # Listener 管理
└── service.go                   # 服务管理
```

**4. 包功能模块 (重点)**
```
teamserver/pkg/
├── agent/                       # Agent 逻辑 ⭐⭐⭐
│   ├── agent.go                 # Agent 结构定义
│   ├── demons.go                # Demon Agent 实现
│   └── commands.go              # 命令处理
│
├── handlers/                    # 通信处理器 ⭐⭐
│   ├── http.go                  # HTTP/HTTPS Listener
│   ├── smb.go                   # SMB Listener
│   └── external.go              # External C2
│
├── events/                      # 事件系统
│   ├── events.go                # 事件总线
│   ├── demons.go                # Agent 事件
│   └── teamserver.go            # 服务端事件
│
├── db/                          # 数据库操作
│   ├── db.go                    # 数据库初始化
│   └── agents.go                # Agent 数据存储
│
├── packager/                    # 数据包封装
├── common/                      # 通用工具
│   ├── packer/                  # 数据打包
│   ├── crypt/                   # 加密
│   └── parser/                  # 数据解析
│
└── logr/                        # 日志系统
```

**学习重点:**
- `teamserver.go` 如何初始化 WebSocket 和 HTTP 服务
- `dispatch.go` 如何处理来自 Client 和 Agent 的消息
- `handlers/http.go` 如何处理 Agent 的回调 (check-in)
- `agent/demons.go` Demon Agent 的注册和命令处理

---

### 第三阶段：Demon Agent 客户端 (5-7 天)

**5. Agent 核心架构 ⭐⭐⭐**
```
payloads/Demon/
├── include/
│   └── Demon.h                  # Agent 主结构体定义 ⭐
│
├── src/
│   ├── Demon.c                  # 入口和主循环 ⭐⭐⭐
│   ├── main/
│   │   └── Main.c               # 初始化代码
│   │
│   ├── core/                    # 核心功能
│   │   ├── Transport.c          # HTTP/SMB 通信 ⭐⭐
│   │   ├── Package.c            # 数据包封装/解析
│   │   ├── Syscalls.c           # 间接系统调用 ⭐⭐
│   │   ├── Win32.c              # WinAPI 动态解析
│   │   ├── Jobs.c               # 任务管理
│   │   ├── Token.c              # Token 操作
│   │   ├── Pivot.c              # SMB Pivot
│   │   ├── CoffeeLdr.c          # BOF 加载器
│   │   └── HwBpEngine.c         # 硬件断点 (AMSI/ETW)
│   │
│   ├── crypt/                   # 加密
│   │   └── AesCrypt.c           # AES 加密通信
│   │
│   └── inject/                  # 注入技术
│       └── Inject.c             # 进程注入
│
└── include/core/
    ├── Transport.h
    ├── Package.h
    ├── Syscalls.h
    └── ...
```

**关键数据结构:**
- `INSTANCE` 结构体 (Demon. H) - 全局状态管理
- `PACKAGE` - 通信数据包格式
- `PHOST_DATA` - C 2 服务器配置

**学习重点:**
1. **初始化流程**: `DemonInit()` → `DemonConfig()` → `DemonRoutine()`
2. **通信流程**: `TransportSend()` → HTTP POST → `TransportRecv()`
3. **命令执行**: 接收 PACKAGE → 解析命令 ID → 执行 → 返回结果
4. **安全技术**:
   - 间接系统调用 (Indirect Syscalls) 实现
   - Sleep Obfuscation 机制 (Ekko/Zilean/FOLIAGE)
   - AMSI/ETW 绕过 (硬件断点)
   - 代理加载 (Proxy Loading)

---

### 第四阶段：Client 客户端 (3-4 天)

**6. UI 客户端代码**
```
client/
├── src/
│   ├── Main.cc                  # 程序入口
│   ├── global.cc                # 全局定义
│   │
│   ├── UserInterface/
│   │   ├── HavocUi.cc           # 主 UI 初始化 ⭐
│   │   ├── Dialogs/             # 对话框
│   │   │   ├── Connect.cc       # 连接对话框
│   │   │   ├── Listener.cc      # Listener 配置
│   │   │   └── Payload.cc       # 生成 Payload
│   │   │
│   │   ├── Widgets/
│   │   │   ├── SessionView.cc   # Session 视图 ⭐
│   │   │   ├── TeamserverTab.cc # Teamserver 标签页
│   │   │   └── ...
│   │   │
│   │   └── SmallWidgets/        # 小组件
│   │
│   └── Util/                    # 工具函数
│
└── include/
    └── UserInterface/
        └── HavocUI.hpp
```

**学习重点:**
- WebSocket 连接和认证流程
- Session 管理和 UI 展示
- 命令发送和结果接收

---

### 第五阶段：高级主题 (可选)

**7. 拓展功能**
```
├── payloads/Shellcode/          # Shellcode 模板
├── payloads/DllLdr/             # DLL 加载器
├── teamserver/pkg/handlers/     # External C2
└── profiles/                    # 配置文件
```

**8. 数据流分析**
```
命令下发流程:
Client UI → WebSocket → Teamserver dispatch → HTTP Handler → Demon Agent

结果返回流程:
Demon Agent → HTTP POST → Teamserver → WebSocket → Client UI
```

---

## 🎯 核心学习建议

### 1. 先理解通信协议
- 查看 `pkg/packager/` 了解数据包格式
- 查看 `pkg/common/packer/` 了解打包机制
- 理解 AES 加密通信流程

### 2. 重点文件优先级
| 优先级 | 文件 | 说明 |
|--------|------|------|
| P 0 | `payloads/Demon/src/Demon.c` | Agent 主循环 |
| P 0 | `payloads/Demon/include/Demon.h` | Agent 结构体 |
| P 0 | `teamserver/cmd/server/dispatch.go` | 消息分发 |
| P 0 | `teamserver/pkg/handlers/http.go` | HTTP 通信 |
| P 1 | `teamserver/pkg/agent/demons.go` | Agent 管理 |
| P 1 | `payloads/Demon/src/core/Transport.c` | 传输层 |
| P 1 | `payloads/Demon/src/core/Package.c` | 数据包处理 |
| P 2 | `payloads/Demon/src/core/Syscalls.c` | 系统调用 |
| P 2 | `client/src/UserInterface/HavocUi.cc` | UI 界面 |

### 3. 调试建议
- Teamserver 使用 `--debug` 模式运行
- 查看 `teamserver/pkg/logr/` 了解日志系统
- 使用 Wireshark 分析 HTTP 通信

---

## 📖 辅助资料

- **Wiki**: https://havocframework.com/
- **GitHub**: https://github.com/HavocFramework/Havoc
- **Discord**: 社区讨论

希望这份指南能帮助你系统地学习 Havoc 框架！建议按阶段逐步深入，先从 Teamserver 和 Agent 的通信流程开始理解。
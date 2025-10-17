# 🚀 enginelabs-2api (凤凰版 v4.0) - 您的终极 cto.new 智能代理

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Version](https://img.shields.io/badge/Version-v4.0_Phoenix-brightgreen)](https://github.com/lzA6/enginelabs-2api)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue?logo=docker)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)](https://www.python.org/)
[![Status](https://img.shields.io/badge/Status-稳定运行-green)](https://github.com/lzA6/enginelabs-2api)

> **项目核心哲学** 💖
>
> 我们相信，技术的魅力不仅在于其强大，更在于其能够被轻松地分享、理解和使用。`enginelabs-2api` 不仅仅是一堆代码，它是一种宣言：**"让强大的 AI 能力，丝滑地融入每一个人的工作流中。"**
>
> 它像一位不知疲倦的信使，为你打通 `cto.new` (EngineLabs) 与 OpenAI 生态之间的壁垒。它鼓励你动手，去探索，去创造。当你运行它的那一刻，你不仅仅是一个使用者，你是一位开拓者，一位将先进生产力掌握在自己手中的魔法师。我们希望这份文档能点燃你的热情，让你感受到开源的快乐和创造的价值。**你，就是改变的开始！**

---

## ✨ 项目简介

`enginelabs-2api` 是一个高性能、轻量级的代理服务。它的核心使命只有一个：

**将 [cto.new](https://cto.new) (原 EngineLabs) 的原生 API，无缝转换为与 OpenAI API 完全兼容的格式。**

这意味着，你现在可以：

- 把你最喜欢的、支持 OpenAI 接口的第三方客户端（比如 [NextChat](https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web), [LobeChat](https://github.com/lobehub/lobe-chat) 等）直接对准本项目
- 享受 `cto.new` 提供的强大模型（如 `ClaudeSonnet4_5`, `GPT5` 等）
- 彻底告别手动刷新、获取和管理 `token` 的繁琐过程！本项目实现了 **"全自动令牌续期"**，一次配置，永久省心

### 🎯 核心特性

#### ✅ 优势亮点
- **一劳永逸**：独创的 `CLERK_COOKIE` 认证机制，实现令牌全自动续期，部署后无需任何人工干预
- **高度兼容**：完美模拟 OpenAI 的 `/v1/chat/completions` 和 `/v1/models` 接口，支持流式响应
- **极致性能**：基于 FastAPI 和 Uvicorn，异步 I/O 处理，响应迅速
- **智能对话**：内置对话历史缓存，能有效利用 `cto.new` 的上下文记忆功能，让对话更连贯
- **部署简单**：Docker-Compose 一键启动，小白也能在 3 分钟内完成部署
- **轻量可靠**：占用资源极低，适合个人服务器或 NAS 长期运行

#### ⚠️ 注意事项
- **依赖上游**：项目的稳定性强依赖于 `cto.new` 和 Clerk 的前端接口
- **Cookie 初始获取**：首次配置需要用户从浏览器手动获取 `CLERK_COOKIE`
- **缓存非持久**：当前的对话历史缓存是基于内存的，服务重启后会丢失

---

## 🏗️ 系统架构

### 🔄 数据流架构图

```
┌─────────────────┐    HTTP Request    ┌──────────────────┐    WebSocket    ┌─────────────────┐
│                 │ ─────────────────> │                  │ ───────────────> │                 │
│   OpenAI Client │                    │  enginelabs-2api │                  │   cto.new API   │
│ (NextChat, etc.)│ <───────────────── │   (本代理服务)    │ <─────────────── │   (上游服务)    │
│                 │   SSE Stream       │                  │   Real-time      │                 │
└─────────────────┘                    └──────────────────┘   Data Stream    └─────────────────┘
         │                                      │                                      │
         │                                      │                                      │
         │                                      │                                      │
         │                              ┌───────▼───────┐                      ┌───────▼───────┐
         │                              │   Clerk Auth  │                      │ 对话历史管理   │
         │                              │   (认证服务)   │                      │ (上下文缓存)   │
         │                              └───────────────┘                      └───────────────┘
```

### 🔧 核心工作流程

1. **认证流程**：`CLERK_COOKIE` → JWT Token 自动续期
2. **请求转换**：OpenAI 格式 → cto.new 格式
3. **上下文管理**：对话历史缓存与复用
4. **流式响应**：WebSocket 实时数据 → SSE 格式转换

---

## 📁 项目结构

```bash
enginelabs-2api/
├── 📄 .env                    # 环境配置文件（部署时创建）
├── 📄 .env.example            # 环境配置模板
├── 📄 Dockerfile              # Docker 镜像构建配置
├── 📄 docker-compose.yml      # Docker 容器编排
├── 📄 main.py                 # FastAPI 应用主入口
├── 📄 nginx.conf              # Nginx 反向代理配置
├── 📄 requirements.txt        # Python 依赖清单
└── 📂 app/
    ├── 📂 core/
    │   ├── 📄 __init__.py
    │   └── 📄 config.py       # 应用配置管理
    ├── 📂 providers/
    │   ├── 📄 __init__.py
    │   ├── 📄 base_provider.py     # 服务提供者基类
    │   └── 📄 enginelabs_provider.py # EngineLabs 核心交互逻辑
    └── 📂 utils/
        └── 📄 sse_utils.py    # Server-Sent Events 工具函数
```

---

## 🧠 工作原理详解

### 🎭 角色扮演版（通俗理解）

想象一下，你想进入一个名为 `cto.new` 的魔法城堡，但你只有一个普通的 OpenAI 通行证。`enginelabs-2api` 就是你的专属魔法翻译官和向导：

1. **出示永久通行证**：你把特殊的"永久通行证" (`CLERK_COOKIE`) 交给向导
2. **自动换取临时门票**：向导自动用通行证换取有时效的"临时门票" (`JWT Token`)
3. **翻译你的请求**：向导将 OpenAI 标准语言翻译成城堡能理解的格式
4. **保持对话连贯**：向导使用"记忆水晶" (`chatHistoryId`) 记住之前的对话
5. **实时传话响应**：通过"传声管道" (`WebSocket`) 实时接收并翻译智者的回答

### 🔬 技术实现版

#### 1. 认证与令牌管理 (`_get_fresh_jwt`)
```python
# 核心流程
cookie → Clerk API → JWT Token → 用户身份验证
```

- 使用 `cloudscraper` 绕过 Cloudflare 保护
- 自动调用 Clerk 令牌刷新接口
- 返回包含用户身份信息的新鲜 JWT

#### 2. 上下文缓存管理 (`_get_conversation_fingerprint`)
```python
# 对话指纹生成
history_messages → JSON序列化 → MD5哈希 → fingerprint
```

- 基于对话历史生成唯一指纹
- 内存缓存 `fingerprint → chatHistoryId` 映射
- 实现对话上下文的智能复用

#### 3. 请求转换与流式处理
```python
# 完整处理流程
1. 接收 OpenAI 格式请求
2. 获取新鲜 JWT 令牌
3. 生成/获取对话历史 ID
4. 发送触发请求到 cto.new
5. 建立 WebSocket 连接接收流式响应
6. 实时转换为 SSE 格式返回客户端
```

---

## 🚀 快速开始

### 📋 前置要求

- **Docker** & **Docker Compose**
- **Git**
- 可访问 `cto.new` 的网络环境

### ⚡ 三分钟部署指南

#### 第一步：获取项目代码
```bash
git clone https://github.com/lzA6/enginelabs-2api.git
cd enginelabs-2api
```

#### 第二步：配置环境变量
```bash
# 复制配置模板
cp .env.example .env

# 编辑配置文件
nano .env
```

配置内容示例：
```env
# 设置你的 API 密钥（用于客户端认证）
API_MASTER_KEY=your_super_secure_api_key_here

# 服务端口（默认 8089）
NGINX_PORT=8089

# 从浏览器获取的 Clerk Cookie（关键配置）
CLERK_COOKIE="__client=...; __session=...; ..."
```

#### 🔍 如何获取 CLERK_COOKIE

1. 在 Chrome/Edge 中登录 [cto.new](https://cto.new)
2. 按 `F12` 打开开发者工具
3. 切换到 **Network** 标签页
4. 在页面中进行任意操作（如发送消息）
5. 找到发往 `clerk.cto.new` 或 `cto.new` 的请求
6. 在 **Request Headers** 中找到 `cookie` 字段
7. **右键点击 → Copy value** 复制完整内容

#### 第三步：启动服务
```bash
# 一键部署
docker-compose up -d

# 查看日志确认运行状态
docker-compose logs -f
```

#### 第四步：配置客户端

在支持 OpenAI 的客户端中配置：

- **API 地址**: `http://你的服务器IP:8089`
- **API 密钥**: 在 `.env` 中设置的 `API_MASTER_KEY`
- **模型**: 选择 `ClaudeSonnet4_5` 或 `GPT5` 等 cto.new 支持的模型

---

## 🛠️ 技术栈深度解析

| 技术组件 | 应用场景 | 复杂度 | 优势 | 替代方案 |
|---------|---------|--------|------|----------|
| **FastAPI** | Web 服务框架 | ★★☆☆☆ | 高性能、异步、自动文档 | Flask, Django |
| **Docker** | 容器化部署 | ★★★☆☆ | 环境隔离、一键部署 | 宿主机部署 |
| **Cloudscraper** | 反爬虫绕过 | ★★★☆☆ | 智能绕过 Cloudflare | requests, httpx |
| **WebSockets** | 实时通信 | ★★★★☆ | 低延迟、双向通信 | 长轮询、SSE |
| **JWT** | 身份认证 | ★★☆☆☆ | 无状态、标准化 | Session, OAuth |
| **Nginx** | 反向代理 | ★★★☆☆ | 高性能、负载均衡 | Caddy, Traefik |
| **异步生成器** | 流式响应 | ★★★★☆ | 内存友好、实时性 | 同步响应 |

### 🔧 关键技术实现

#### 1. 异步流式处理
```python
async def stream_generator(messages, model):
    # 获取认证令牌
    jwt_token = await self._get_fresh_jwt()
    
    # 管理对话上下文
    chat_history_id = await self._get_conversation_fingerprint(messages)
    
    # 建立 WebSocket 连接
    async with websockets.connect(ws_url) as websocket:
        while True:
            message = await websocket.recv()
            # 实时转换格式并返回
            yield format_to_sse(message)
```

#### 2. 智能缓存机制
```python
def _get_conversation_fingerprint(self, messages):
    """基于对话历史生成唯一指纹"""
    if len(messages) <= 1:
        return str(uuid.uuid4())
    
    history_str = json.dumps(messages[:-1], sort_keys=True)
    fingerprint = hashlib.md5(history_str.encode()).hexdigest()
    
    if fingerprint in self.conversation_cache:
        return self.conversation_cache[fingerprint]
    else:
        new_id = str(uuid.uuid4())
        self.conversation_cache[fingerprint] = new_id
        return new_id
```

---

## 📈 项目演进路线

### ✅ 已实现功能 (v4.0 凤凰版)

- [x] **核心代理功能**：完整的 chat/completions 代理和流式响应
- [x] **自动认证系统**：CLERK_COOKIE → JWT 自动续期
- [x] **上下文管理**：基于内存的对话历史缓存
- [x] **容器化部署**：Docker + Docker Compose 完整支持
- [x] **模型列表接口**：兼容 OpenAI 的 `/v1/models` 端点

### 🔄 当前局限性与改进方向

#### 1. 缓存持久化 (优先级：高)
**问题**：内存缓存重启丢失，影响上下文连贯性  
**解决方案**：集成 Redis 实现持久化存储

#### 2. 错误处理增强 (优先级：高)
**问题**：网络异常和上游服务变更处理不够健壮  
**解决方案**：实现指数退避重试和熔断机制

#### 3. 动态模型发现 (优先级：中)
**问题**：模型列表硬编码，无法自动发现新模型  
**解决方案**：通过逆向工程获取动态模型列表

### 🚀 未来发展规划

#### 短期目标 (1-2个月)
1. **Redis 集成**：实现持久化缓存存储
2. **健康检查**：完善的服务监控和自愈机制
3. **日志系统**：结构化日志和日志轮转

#### 中期目标 (3-6个月)
1. **多账户支持**：支持多个 CLERK_COOKIE 轮换使用
2. **配置界面**：Web 管理界面简化配置
3. **插件体系**：可扩展的插件架构

#### 长期愿景
1. **多云支持**：适配多个 AI 服务平台
2. **集群部署**：支持高可用分布式部署
3. **生态建设**：建立完整的工具链和社区

---

## 🛡️ 故障排除与维护

### 🔍 常见问题

#### Q1: 服务启动失败
**症状**：Docker 容器无法正常启动  
**排查**：
```bash
# 查看详细日志
docker-compose logs

# 检查环境变量配置
cat .env | grep -v "COOKIE"
```

#### Q2: 认证失败
**症状**：API 返回 401 错误  
**解决**：
1. 重新获取 CLERK_COOKIE（可能已过期）
2. 确认 Cookie 格式正确（包含在双引号中）

#### Q3: 流式响应中断
**症状**：对话中途断开连接  
**排查**：
- 检查网络稳定性
- 查看服务端日志确认错误信息

### 🔄 日常维护

#### 监控服务状态
```bash
# 查看服务运行状态
docker-compose ps

# 实时查看日志
docker-compose logs -f --tail=50
```

#### 更新服务
```bash
# 拉取最新代码
git pull origin main

# 重新构建并重启服务
docker-compose down
docker-compose up -d --build
```

---

## 🤝 贡献指南

我们欢迎各种形式的贡献！无论是代码改进、文档完善，还是问题反馈，都是对项目的宝贵支持。

### 贡献方式

1. **报告问题**：在 GitHub Issues 中反馈 bug 或建议
2. **代码贡献**：Fork 项目并提交 Pull Request
3. **文档改进**：帮助完善文档和教程
4. **社区分享**：分享使用经验和最佳实践

### 开发环境搭建

```bash
# 克隆项目
git clone https://github.com/lzA6/enginelabs-2api.git
cd enginelabs-2api

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate  # Windows

# 安装依赖
pip install -r requirements.txt

# 配置环境变量
cp .env.example .env
# 编辑 .env 文件配置参数

# 启动开发服务器
uvicorn main:app --reload --host 0.0.0.0 --port 8089
```

---

## 📜 开源协议

本项目采用 **Apache License 2.0** 开源协议。

### 协议要点

- **允许**：商业使用、修改、分发、专利授权
- **要求**：保留版权声明和许可声明
- **禁止**：使用项目名称进行背书
- **无担保**：作者不承担使用风险

完整的协议文本请参阅 [LICENSE](LICENSE) 文件。

---

## 🌟 致谢

感谢所有为这个项目做出贡献的开发者、测试者和文档编写者。特别感谢：

- `cto.new` 团队提供的优秀 AI 服务
- OpenAI 建立的 API 标准生态
- 开源社区提供的各种优秀工具和库

---

**愿代码与你同在，愿创造的喜悦永远伴随你！**  
**May the source be with you!**

---

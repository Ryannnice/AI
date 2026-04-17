# **稔远的个人学习要点；仅个人纪录供参考，不含专业知识细节**


# ***2025.3.19***

## 知识学习
### 图解Transformer 
非常清晰的图示教程，用矩阵拆解Transformer，难度梯度很合适
[教程](https://jalammar.github.io/illustrated-transformer/)

### TypeScript/npm包管理

#### TypeScript[(清华大学的AI教育项目)](https://github.com/Ryannnice/CUHK-LLM-Edu/edit/main/README.md)

整个项目绝大部分使用TypeScript,文件结构、调用逻辑非常复杂
TypeScript 是 JavaScript 的超集

#### npm

npm = Node Package Manager（Node.js 包管理器）

安装库和工具/管理依赖（项目里用到的第三方包）/**运行脚本**（比如启动 Vue、React 或 Node 项目）
*npm run <脚本名>: 运行 package.json 里的脚本*

### FastAPI python框架了解
第一次实习，第一次接触偏工程的项目：不同于科研的是，可行性/整体模块的缝合、运行似乎比细致的优化更重要

清华的开源教育项目未采用前后端分离架构

使用.json格式请求体完成数据/信息传递

前端直接通过函数调用REST API:
```
 fastapi_backend/                                           
    └── static/                                              
        ├── index.html        # 主页（需求输入 + 设置）      
        ├── generate.html     # 生成流程页（SSE 大纲流 + 进度）
        ├── classroom.html    # 课堂播放页（幻灯片/测验/聊天）
        ├── app.js            # 全局工具：API 调用、设置存储、路由                                     
        ├── generate.js       # 生成流程逻辑                 
        ├── classroom.js      # 课堂播放逻辑（幻灯片渲染、测验、聊天）                   
        └── style.css         # 全局样式     
```


## 实践

### Fast API
在原本的原本的REST API接口上，建立/fastapi_backend文件夹，用Fast API封装全部18个功能的api
原有.ts文件前端直接调用api的所有逻辑均保留，与Fast后端接口不冲突

实现后端分离之后，建立/fastapi_backend/static文件夹，仅使用js/html初步实现前端功能，以验证Fast API后端接口可行性


# ***2025.3.20***

## 知识学习

### 开源库OnlySpecs
这是自动生成软件的agent系统，可能对项目第二部分*WorkShop*有帮助
（上午团队实现workshop功能时发现直接调用LLM实现代码（软件编程）能力有限：贪吃蛇不成功，推箱子成功）
试图部署该开源项目，接入我们的项目


## 实践

### Linux bash
上午claude api爆了，以为是网络问题重新配置安装一遍windows WSL的linux的网络环境


# ***2025.3.21***

## 知识学习

### 开源库OnlySpecs

#### node-pty

pty.spawn("claude")

相当于**在程序里打开**一个**终端**窗口

#### AIEngine 

一个“可以驱动 Claude CLI 干活”的执行器

```
return new Promise((resolve, reject) => {
    proc.onExit((e) => {
        if (e.exitCode === 0) resolve()
        else reject(new Error(...))
    })
})
```
```
┌─────────────┐
│ run() 调用  │
│ await engine│
└─────┬───────┘
      │
      ▼
┌─────────────┐
│ Promise     │   <-- pending 状态
│ resolve/reject 内部管子
└─────┬───────┘
      │
      ▼
┌─────────────┐
│ proc.onExit │  <-- Claude CLI 退出触发
│ e.exitCode  │
└─────┬───────┘
      │
      ▼
if(exitCode==0) resolve()  else reject(error)
      │
      ▼
Promise 状态变更 → 外层 await/then/catch 收到结果
```

#### Shim 

是一种兼容层或适配器，用于在不修改原有代码的情况下，让新旧接口或系统之间能够协同工作

这很适用于最小化更改，让该开源项目快速应用于我们的项目中，以此为起点吧

#### Node.js Web 服务器

Node.js 是一个运行环境，可以用 JavaScript 写服务端程序

"Node.js Web 服务器"就是用 Node.js 写的 HTTP 服务，比如用 Express、Fastify、Koa 等框架搭建的后端，和阿里云服务器不冲突

```
阿里云 ECS（服务器硬件/系统）
    └── Nginx（反向代理，监听 80/443 端口）
          └── Node.js 进程（监听 3000 端口）
                └── 你的业务代码      
```


## 实践

### Web版OnlySpecs功能测试

已完成在web上的部署，使用简单的html转跳

汉化前端菜单栏

### 融入大项目的Workshop部分：

**我们项目Workshop的原架构：**

```
用户 → Vue
        │
        ▼
FastAPI /generate
        │
        ▼
DeepSeek 生成 HTML
        │
        ▼
FastAPI /upload
        │
        ▼
阿里云 OSS
        │
        ▼
返回 URL
        │
        ▼
Vue 展示
```

**开源软件OnlySpecs的原架构：**
```
Electron UI
    ↓
Renderer (DOM + Monaco)
    ↓
IPC
    ↓
Main Process
    ↓
node-pty
    ↓
Claude CLI
```

**新架构融合，两种方案：**

***方案一，分离式：***
```
 大项目
 ├── Vue 仪表盘（前端）   → Docker: Nginx 静态托管，端口 80
 ├── FastAPI 后端         → Docker: Uvicorn，端口 9000
 │   ├── /generate        → DeepSeek 流式生成 HTML
 │   └── /upload          → 阿里云 OSS 上传
 └── OnlySpecs（待加入）  → Docker: Node.js，端口 3579
     └── 功能：Specs 编写、Claude AI 代码生成、终端
```

***方案二，通过Fast API使用功能，仅替换掉LLM，使用claude agent编写软件：***
```
Vue 前端
    ↓ POST /generate-software { prompt }
  FastAPI
    ↓ 调用 OnlySpecs Node.js 服务（HTTP 或子进程）
  OnlySpecs Web Server
    ↓ 写 specs.md → 启动 Claude CLI
  Claude CLI（node-pty）
    ↓ 生成代码
  返回结果（文件路径 / OSS URL）
    ↑ 流式进度推送（SSE / WebSocket）
  Vue 前端展示


FastAPI 端点设计
  # POST /generate-software
  # 输入：用户 prompt
  # 输出：SSE 流式进度 + 最终代码 URL

  @app.post("/generate-software")
  async def generate_software(prompt: str):
      # 1. 调用 OnlySpecs API 创建 specs 文件
      # 2. 触发 Generate from Specs
      # 3. 流式返回进度
      # 4. 完成后上传到 OSS，返回 URL
```

先尝试方案二，先设计无头OnlySpecs的API




# ***2025.3.22***

## 知识学习


## 实践

Fast API 编写完成，核心是/generate 根据用户指令来交给OnlySpecs，利用其功能生成

api测试成功（文档：/home/ryan/OnlySpecs/docs/API_QUICKSTART.md，测试：终端运行 npm run test:api）

接下来对接我们的项目第二部分Workshop：
实现方式参考原框架，写出仿制的前端：/home/ryan/OnlySpecs/api-integration

整个Pipeline:

```
  📁 Project Structure        

  ~/OnlySpecs/api-integration/                   
  ├── app.py              # FastAPI backend (API proxy + SSE streaming)
  ├── requirements.txt    # Python dependencies          
  ├── .env               # Environment configuration         
  ├── .env.example       # Environment template           
  ├── start.sh           # Quick start script                                            
  ├── static/                                                                                             
  │   └── index.html     # Vue 3 frontend (312 lines)
  └── README.md          # Complete documentation                                                                                               
                                                                                                                                                
  🎯 Key Features Implemented

  Backend (FastAPI):                      
  - ✅ CORS-enabled API proxy to OnlySpecs API  
  - ✅ SSE streaming for real-time log updates
  - ✅ Endpoints: /api/generate, /api/status, /api/logs, /api/tasks, /api/download
  - ✅ Error handling and timeout controls                      
                                                                                                                                                
  Frontend (Vue 3 + Tailwind):         
  - ✅ Clean, responsive UI with Chinese localization
  - ✅ Real-time log display with auto-scroll
  - ✅ Task status tracking (pending/running/completed/failed)
  - ✅ History task list with click-to-load        
  - ✅ Download generated code as ZIP                    
  - ✅ EventSource for SSE log streaming           
                                                                                                                                                
  🚀 Quick Start                   

  # 1. Start OnlySpecs API (in one terminal)  
  cd ~/OnlySpecs      
  npm run api            

  # 2. Start frontend (in another terminal)                       
  cd ~/OnlySpecs/api-integration  
  ./start.sh                          
                                                                                                    
  Then visit: http://localhost:9000     

  📝 Usage Flow                            
  1. Enter software requirements in the text area                                                                                               
  2. Click "开始生成" (Start Generation)                  
  3. Watch real-time Claude CLI logs                            
  4. Download code when complete or open in file explorer
  5. View history tasks in the collapsible section                  
                                                                                                                                                
  The implementation follows the plan exactly, using SSE for real-time updates and providing a simple, user-friendly interface for interacting  
  with OnlySpecs.    
```

已经能完美运行，依靠简洁的web界面，通过Fast API和OnlySpecs交互

***输入-->OnlySpecs-->Claude CLI-->输出***，用户只负责敲几个字：项目第二部分低代码的思想

debug修复内容：

1. claude开始但是不工作，代码写不进去项目文件夹 / claude 不动，接收不到指令：--print 标志可以完全跳过交互式 UI，直接输出结果。不需要 pty 模拟，改用子进程即可。用 spawn + --print 替换整个 pty 方案，彻底解决交互式 UI 问题。

2. 下载 ZIP之后win系统打不开：之前是把 OnlySpecs API 返回的 JSON 当 ZIP 存的，当然打不开。现在后端拿到 codePath，用 shutil.make_archive 真正打包成 ZIP，Win11 可以直接解压。

3. “在文件管理器中打开”的按钮点不动：新增了 /api/open/{task_id} 接口，调用 xdg-open 打开 Linux 文件管理器，同时在界面显示代码路径。
WSL2 里 xdg-open 无法直接打开 Windows 文件管理器。需要用 explorer.exe 来打开，但路径要转换成 Windows 格式。
转换出来是 \\wsl.localhost\Ubuntu\... 格式，Win11 的文件资源管理器可以直接打开这个 UNC 路径。
```
\\wsl.localhost\Ubuntu\home\ryan\Documents\OnlySpecs\api-workspaces\task_1774168877875_1u1yudaz6\code_v0001
```

***全部修复***

朝着更更更低代码平台进发：


📋 计划总结

核心功能： 在 Web 界面添加 4 种输出类型选择：

1. 📄 源代码 - 可编辑的源文件
2. 🌐 Web 应用 - 单文件 HTML，浏览器直接运行
3. 💻 桌面程序 - Windows .exe 可执行文件（自动打包）
4. 📱 手机应用 - PWA 渐进式 Web 应用

用户使用Pipeline:

```
用户浏览器
    │
    ▼
Vue 前端
    │
    ▼
FastAPI API
    │
    ▼
OnlySpecs AI Engine （调用Claude CLI）
    │
    ▼
生成代码
    │
    ▼
Build Worker（Docker）
    │
    ▼
编译为 .exe
    │
    ▼
上传到 OSS
    │
    ▼
用户下载
```

本地项目列表已更新：

新功能：
1. 类型标签 - 每个项目左侧显示彩色标签：
  - 📄 源代码（紫色）
  - 🌐 Web应用（靛蓝色）
  - 💻 桌面程序（橙色）
  - 📱 手机应用（粉色）
2. 对应按钮 - 根据项目类型显示不同的主功能按钮：
  - 源代码：📄 线上查看 + 📦 下载源码
  - Web应用：🌐 线上运行 + 📦 下载源码
  - 桌面程序：💻 下载 .exe + 📦 下载源码
  - 手机应用：📱 下载应用 + 📦 下载源码


```
  部署步骤：

  1. 在阿里云服务器上安装依赖：
  安装Node.js
  curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
  sudo apt-get install -y nodejs

  安装Python
  sudo apt-get install python3 python3-pip

  安装Docker
  curl -fsSL https://get.docker.com | sudo sh
  sudo usermod -aG docker $USER

  安装Claude CLI
  npm install -g @anthropic-ai/claude-code

  2. 配置Claude API密钥：
  export ANTHROPIC_API_KEY="your-api-key-here"
                                              
  3. 打包项目：                              
  cd ~/OnlySpecs                            
  tar -czf onlyspecs-deploy.tar.gz \ 
    src/ \                                  
    api-integration/ \   
    package.json\                          
    tsconfig.json                                                                                                                             
  4. 上传到服务器：                         
  scp onlyspecs-deploy.tar.gz user@your-server:/home/user/      

  5. 在服务器上解压并启动：                 
  tar -xzf onlyspecs-deploy.tar.gz                               
  npm install                               
  docker pull cdrx/pyinstaller-windows        
                                           
  启动服务                                 
  npm run api &                             
  cd api-integration && python3 app.py &                               

  6. 配置防火墙：
  sudo ufw allow 3580                                   
  sudo ufw allow 9000                       

  访问：http://your-server-ip:9000 
```




# ***2025.3.23***

## 知识学习

### Docker

```
https://www.bilibili.com/video/BV1THKyzBER6/?share_source=copy_web&vd_source=035cd776909e96dadfc9bbaeb1588cd4
```

### Fast API

Fast API配合**后端开发**：
```
https://www.bilibili.com/video/BV1eUxve7Ein/?share_source=copy_web&vd_source=035cd776909e96dadfc9bbaeb1588cd4
```
```
https://fastapi.org.cn/python-types/#pydantic-models
```

###

上下文工程 Context Engineering

### 谕书的Workshop框架和API：

项目框架采用FC容器(每个项目一个)，而不是fs操作本地文件
```
OSS+FC的架构：
┌─────────────────────────────────────────────────────┐
│                    用户浏览器                         │
│  左：对话+文件树    中：代码查看    右：预览iframe      │
└──────────┬──────────────────────────────┬───────────┘
           │              │ iframe src
           ▼                             ▼
┌─────────────────┐            ┌──────────────────────┐
│  你的FastAPI服务 │            │  FC容器(每个项目一个) │
│  (项目管理/调度) │            │   dev server:9000    │
└────────┬────────┘            └──────────┬───────────┘
         │                                │
    ┌────┴──────────────────────────┐     │
    │         阿里云基础设施         │     │
    │  OSS(代码+静态)  RDS(MySQL)    |◄───┘
    │  ACR(镜像仓库)   MNS(消息队列) │
    └───────────────────────────────┘
```
这个线上部署运行架构值得融合，今天做这件事：

## 实践


```
OnlySpecs 的现有架构梳理：       

  四种运行模式:                            
  ┌──────────────┬───────────────┬──────┬─────────────────────┐                                                                             
  │     模式     │     入口      │ 端口 │        说明         │                                                                             
  ├──────────────┼───────────────┼──────┼─────────────────────┤                                                                             
  │ 桌面应用     │ npm run dev   │ —    │ Electron + IPC      │                                                                             
  ├──────────────┼───────────────┼──────┼─────────────────────┤                                                                             
  │ 无头 API     │ npm run api   │ 3580 │ Express + 子进程    │                                                                             
  ├──────────────┼───────────────┼──────┼─────────────────────┤                                                                             
  │ Web 界面     │ npm run web   │ 3579 │ Express + WebSocket │                                                                             
  ├──────────────┼───────────────┼──────┼─────────────────────┤                                                                             
  │ FastAPI 前端 │ python app.py │ 9000 │ 代理 API + SSE      │
  └──────────────┴───────────────┴──────┴─────────────────────┘                                                                       
  核心数据流:

  桌面模式：
  Renderer (Monaco/xterm) → IPC → ipc-handlers.ts → 文件系统 / node-pty / Claude SDK         

  API模式：                               
  HTTP POST /generate → 创建任务 → 启动 claude CLI子进程 → 写入 code_v0001/                                                                 
  HTTP GET /logs/:id→ 轮询日志                      

  Web 模式：                                        
  浏览器 → WebSocket → web-server（复用 ipc-handlers 逻辑）→ 文件系统 / node-pty                                                                                                                                  
  FastAPI 模式：                            
  Vue3 → FastAPI (9000) → HTTP → API Server (3580) → claude CLI↑ SSE 实时日志流
  
  关键文件                                  
                                                                                                                                            
  ┌──────────────────────────┬─────────────────────────────────────────────┐
  │           文件           │                    作用                     │                                                                
  ├──────────────────────────┼─────────────────────────────────────────────┤                                                                
  │ src/main/ipc-handlers.ts │ 所有 IPC 处理（编辑器、终端、文件、GitHub） │                                                                
  ├──────────────────────────┼─────────────────────────────────────────────┤                                                                
  │ src/main/claude/sdk.ts   │ Claude Agent SDK 封装，流式输出             │                                                                
  ├──────────────────────────┼─────────────────────────────────────────────┤                                                                
  │ src/renderer/index.ts    │ 主App 类，组装所有 UI组件                   │                                                                
  ├──────────────────────────┼─────────────────────────────────────────────┤                                                                
  │ src/api-server/index.ts  │ 无头 API，任务管理 + 子进程调度             │
  ├──────────────────────────┼─────────────────────────────────────────────┤                                                                
  │ src/web-server/index.ts  │ WebSocket 服务，镜像 IPC 协议               │
  ├──────────────────────────┼─────────────────────────────────────────────┤                                                                
  │ api-integration/app.py   │ FastAPI 代理 + SSE + ZIP 下载 + 项目管理    │
  └──────────────────────────┴─────────────────────────────────────────────┘                                                                
                                                            
  ---                                                                                                                                       
  数据存储                                                  
                                            
  ~/Documents/OnlySpecs/
  ├── editors/          # 编辑器内容（每个 tab 一个 JSON）
  ├── config.json       # API Key、上次项目路径 
  ├── api-workspaces/   # API 模式生成的代码             
  │   └── task_*/                         
  │├── specs_v0001.md                       
  │       └── code_v0001/                 
  └── tmp/              # GitHub import 临时克隆目录    

  两个独立的 Claude 调用路径：                                     
  1. 桌面/Web 模式 — 通过 claude/sdk.ts 调用 @anthropic-ai/claude-agent-sdk，流式返回结果给 UI
  2. API 模式 — 直接 spawn claude CLI 子进程，用 node-pty 捕获输出，存入任务日志                                                            
                                                                                                                                            
  两条路径互相独立，不共享代码。       
```


# ***2025.3.24***

## 知识学习


## 实践

### Debug 网络代理

### 完善数据流向

优化prompt以暴露onlyspecs原有复杂文档生成能力

### Fast API

对接谕书的架构，我第一次跑通了Fast API的agent接口！

已成功完成/generate接口，成功对接、上传到OSS

目前，需完善FC系统


### FC容器

#### 方案规划：
```potential solution:
完成 FC 容器调度与多文件生成验证                       

 Context

 当前状态：
 - Claude CLI 集成完成：POST /generate → Claude CLI → 收集文件 → OSS
 - 标准化 prompt 已实现：build_enhanced_prompt() 注入 scripts/ 目录要求
 - FC API 调度完成：POST /containers/{project_id}/start → 创建 FC 函数 → 返回 preview_url
 - OSS 存储完成：save_project() 上传文件 + manifest.json

 核心问题：
 FC 容器无法启动生成的代码，因为缺少容器内的启动逻辑：
 1. 缺少 entrypoint.sh - 容器启动脚本，负责从 OSS 下载代码并执行标准化脚本
 2. 缺少容器镜像 - base-node18/base-python39/base-fullstack 镜像不存在
 3. 无法验证多文件生成 - 没有端到端测试验证 Claude CLI 生成的复杂项目能否正常运行

 目标：
 完成 FC 调度链路，实现：用户 prompt → Claude 生成 → OSS 存储 → FC 容器运行 → 前端 iframe 预览

 ---
 实现方案

 方案 A：完整 FC 容器方案（生产级）

 需要实现：
 1. 创建 3 个 Dockerfile（base-node18, base-python39, base-fullstack）
 2. 编写 entrypoint.sh 脚本（下载 OSS 代码 → 执行 prepare.sh → 执行 dev.sh）
 3. 构建镜像并推送到 ACR
 4. 验证完整链路

 优点： 真实生产环境，完全符合 goal.md 架构
 缺点： 需要 Docker 环境、ACR 推送权限、FC 配额

 ---
 方案 B：本地验证方案（快速验证）

 只验证文件生成和脚本执行，不依赖 FC：
 1. 调用 /generate 生成项目
 2. 从 OSS 下载生成的文件到本地临时目录
 3. 本地执行 scripts/prepare.sh 和 scripts/dev.sh
 4. 验证服务能在 9000 端口启动

 优点： 快速验证，无需云资源
 缺点： 不是真实 FC 环境

 ---
 推荐方案：方案 B（本地验证）+ 方案 A 的容器脚本准备

 分两步走：

 第一步：本地验证多文件生成（立即可做）

 创建测试脚本 test_generation.py：
 - 调用 /generate API 生成简单项目（如 "创建一个 Hello World 网页"）
 - 检查返回的 files 列表是否包含：
   - scripts/prepare.sh
   - scripts/dev.sh（包含端口 9000）
   - scripts/build.sh
   - scripts/start.sh
   - 业务代码文件（如 src/index.html）
 - 从 OSS 下载所有文件到 /tmp/test-{project_id}/
 - 执行 bash scripts/prepare.sh（安装依赖）
 - 后台执行 bash scripts/dev.sh（启动服务）
 - 验证 curl http://localhost:9000 返回 200

 验证目标： 确认 Claude CLI 生成的项目结构正确，脚本可执行

 ---
 第二步：准备 FC 容器资源（为生产部署做准备）

 创建 3 个容器镜像的 Dockerfile 和 entrypoint.sh：

 文件结构：
 Reference-framework/
   docker/
     base-node18/
       Dockerfile
       entrypoint.sh
     base-python39/
       Dockerfile
       entrypoint.sh
     base-fullstack/
       Dockerfile
       entrypoint.sh
```

#### 本地运行测试：

为了验证代码树的运行可行性。之后再真正在云端FC环境运行代码

先完成**本地测试**，之后“登录 ACR”（ACR 指的是 Alibaba Cloud Container Registry）：

把构建的 Docker 镜像**上传到阿里云的镜像仓库**，让 **FC 容器**可以拉取运行


排查出的三个问题:

问题 1：系统代理拦截了 localhost 请求 → 502               
  
  http_proxy=http://172.26.224.1:7890                       
  所有 requests 发出的请求（包括 localhost）都走了代理，代理无法转发本地流量，返回 502
  解决：对 localhost 请求显式禁用代理，对公网 OSS 保留代理: 
  PROXY_FOR_LOCALHOST = {"http": None, "https": None}
  PROXY_FOR_OSS = {"http": os.environ.get("http_proxy"), ...}
                                       
问题 2：FastAPI 和生成项目端口冲突（都是 9000）

  测试脚本发完请求后，Step 6 验证的 http://localhost:9000 其实打到了 FastAPI 自己
  解决：FastAPI 改用 9001 启动：           
  uvicorn app:app --port 9001

问题 3：prepare.sh 写死 /workspace 本地不存在

  Claude 生成的脚本是为 FC 容器设计的（容器里代码在 /workspace），本地 /workspace 不存在
  解决：下载脚本后自动替换路径： patched = content.replace("/workspace", test_dir)
  
  验证结果:                                 
  ┌───────────────────────────┬────────────────────────────┐                                                                             
  │           步骤            │            结果            │                                                                             
  ├───────────────────────────┼────────────────────────────┤                                                                             
  │ Claude CLI 生成文件       │  7个文件包含所有必需脚本 │                                                                             
  ├───────────────────────────┼────────────────────────────┤                                                                             
  │ OSS 上传/下载             │  正常                    │                                                                             
  ├───────────────────────────┼────────────────────────────┤                                                                             
  │ prepare.sh（npm install） │  成功                    │                                                                             
  ├───────────────────────────┼────────────────────────────┤                                                                             
  │ dev.sh（npx serve :9000） │  成功                    │
  ├───────────────────────────┼────────────────────────────┤                                                                             
  │ HTTP 200 验证             │  返回完整 HTML           │
  └───────────────────────────┴────────────────────────────┘                    
  整条链路 prompt → Claude → OSS → 脚本执行 → HTTP服务 验证完毕。


  端口分配：
  ┌────────────────────────────┬──────┬────────────────────────────┐
  │            用途             │ 端口 │            备注            
  ├────────────────────────────┼──────┼────────────────────────────┤
  │ FastAPI（uvicorn app:app）  │ 9000 │ 正常启动，不变             
  ├────────────────────────────┼──────┼────────────────────────────┤
  │ 本地测试生成项目            │ 8080 │ 脚本自动 patch，测完还原    
  ├────────────────────────────┼──────┼────────────────────────────┤
  │ FC 容器里生成项目           │ 9000 │ 真实环境不 patch，保持原样  
  └────────────────────────────┴──────┴────────────────────────────┘    




# ***2025.3.25***

## 知识学习

### 更全面的Coding agent:

                Workshop UI
                     │
                     ▼
            FastAPI Agent Orchestrator
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     Planner       Coder       Debugger
        │            │            │
        ▼            ▼            ▼
     spec.md      code tree      patch
        │            │            │
        └──────► OSS Storage ◄───┘
                      │
                      ▼
                FC Container
                      │
                      ▼
                Dev Server
                      │
                      ▼
                   iframe

### 实践

Multi-Agnet的使用！震撼！

#### OpenAI Codex, WSL2 CLI 模式运行 

Claude的API实在不稳定，50%时间403/503 error，换用Codex分组：

VS Code的json配置openai codex默认不读取，所以需配置文件：

我使用终端模式在WSL2，配置两个文件后成功运行gpt 5.4 model（默认的gpt 5 拥挤，不行）：

```GNU nano 7.2    /home/ryan/.codex/auth.json
{
  "auth_mode": "apikey",
  "OPENAI_API_KEY": "sk-BfP6tmbEJawv9PzMRdn6nd6OXLbvge72yywVBBbZnfTPvS9V"
}
```

```GNU nano 7.2    /home/ryan/.config/openai/config.toml
disable_response_storage = true
model = "gpt-5-codex"
model_provider = "custom"
model_reasoning_effort = "high"
windows_wsl_setup_acknowledged = true

[model_providers.custom]
base_url = "https://hone.vvvv.ee/v1"
name = "custom"
requires_openai_auth = true
wire_api = "responses"

[notice]
hide_full_access_warning = true
```

#### 优化LLM-CLI

整个调用流程已换成阿里千问百炼

#### 沙箱：

容器实例已经被创建并注册成功，现在要让容器运行起来
核心思想：AI 不直接运行代码，而是在 sandbox container 里运行。       

常规的AI Coding 沙箱架构：
User
  │
  ▼
Workshop UI (Web IDE)
  │
  ▼
Agent Service
  │
  ▼
Task Queue
  │
  ▼
Sandbox Manager
  │
  ▼
Container Runtime
  │
  ▼
Dev Server
  │
  ▼
Preview Router
  │
  ▼
Preview URL
               

流程：
create container
      ↓
git clone / oss pull
      ↓
npm install
      ↓
start dev server
      ↓
expose preview url


	
Response body
Download
{
  "project_ids": [
    "proj-1774117999499",
    "proj-1774118618839",
    "proj-1774118736018",
    "proj-1774118993314",
    "proj-1774119281456",
    "proj-1774119530520",
    "proj-1774261757601",
    "proj-1774334030907",
    "proj-1774336370281",
    "proj-1774345523559",
    "proj-1774365171381",
    "proj-1774365451436",
    "proj-1774365898657",
    "proj-1774366108059",
    "proj-1774366895437",
    "proj-1774367246631",
    "proj-1774402506350",
    "proj-1774403324560",
    "proj-1774424651687",
    "string"
  ]
}

已经添加排查各个项目信息的接口：**GET /projects/summary**
```接口返回：
{
  "projects": [
    {
      "project_id": "proj-1774117999499",
      "manifest_exists": true,
      "template": "base-node18",
      "created_at": "2026-03-21T18:33:20Z",
      "preview_url": "http://snake.fortuneai.cc/projects/proj-1774117999499/index.html",
      "file_count": 12,
      "has_fc": false,
      "fc_status": "NotFound",
      "fc_preview_url": null,
      "manifest_error": null,
      "fc_error": null
    },
```
排查之后，已经逐一删除现存的FC

现在，添加删除某项目的功能

已经全部删除！现在有很多新接口，皆可使用：

```目前的 FastAPI 接口：

POST
/generate
Generate With Openai

POST
/upload
Upload

POST
/save-project
Save Project

GET
/projects
Get Projects

GET
/projects/summary
Get Projects Summary

GET
/projects/{project_id}
Get Project

DELETE
/projects/{project_id}
Delete Project

GET
/projects/{project_id}/tree
Get Project Tree

GET
/projects/{project_id}/file
Get Project File

***container***

POST
/containers/{project_id}/start
Start Container

GET
/containers/{project_id}/status
Get Container Status

DELETE 
/containers/{project_id} 
Delete Container
```

#### 实现较复杂的 Coding Agnet（方案2）:

**Coding Agent方案2：** ***/generate/agent***

把现在的“一次 LLM 调用吐全量文件”升级成
“多轮：生成 -> 本地验证 -> 读错误 -> 修复 -> 再验证”
的闭环，让产物在上传 OSS/跑 FC 前就尽量稳定

架构规划：
把它从“一个接口里把所有事做完”拆成三层，并且用“可组合的工作流 + 可插拔的工具”承载未来功能增长。

  1) 分层与边界（关键）
  - API 层（routers/*）：只负责参数校验、创建任务、返回 run_id/结果，不写业务逻辑。
  - Agent 编排层（agent/*）：状态机 + 工作流引擎。负责多轮循环、停止条件、重试、取消、超时、产物汇总。
  - 工具与基础设施层（tools/* + services/*）：把所有副作用封装成工具接口（LLM、文件系统、命令执行、HTTP 探测、OSS、FC），Agent 只能通过工具做
    事。

  2) 推荐目录结构（可扩展）
  - Reference-framework/routers/agent.py：/generate/agent、/runs/{run_id}、/runs/{run_id}/events、/runs/{run_id}/cancel
  - Reference-framework/agent/core.py：Run 状态机、Step 约定、上下文（Context）
  - Reference-framework/agent/workflows/*.py：工作流定义（生成-only、生成+验证、生成+上传OSS、生成+上传+部署FC…）
  - Reference-framework/agent/steps/*.py：原子步骤（generate_draft、normalize、write_workspace、validate、llm_patch、package、upload_oss、
    deploy_fc…）
  - Reference-framework/agent/tools/*.py：工具接口与实现适配（llm_tool、exec_tool、fs_tool、oss_tool、fc_tool、http_probe_tool）
  - Reference-framework/agent/store/*.py：RunStore/EventStore/ArtifactStore（先本地文件或内存，后续可换 Redis/DB，不影响上层）

  3) 核心数据模型（未来加功能不乱）
  - Run：run_id、status(queued/running/succeeded/failed/canceled)、workflow_name、config、created_at、updated_at
  - RunEvent：按时间追加（step_start/step_end/tool_call/tool_result/log/error），用于调试和前端展示
  - Artifact：生成文件集、workspace 路径、打包产物、OSS key、FC 预览 URL 等（都挂在 run 上）

  4) 工作流设计（把“越来越多功能”变成组合）
  - 每个 workflow 只是一个 step 列表 + 分支条件，例如：
      - generate_only: LLM生成 -> 规范化 -> 返回文件
      - generate_validate: LLM生成 -> 规范化 -> 落盘 -> prepare/build/start -> 健康检查 -> (失败则 patch 循环N次) -> 返回文件+报告
      - generate_save_oss: 在通过验证后追加 upload_oss -> 写manifest
      - generate_save_deploy_fc: 再追加 deploy_fc -> 探测预览URL
  - 每个 step 输入/输出都只读写 Context，不直接互相调用；这样加新能力只要“加 step + 改 workflow 配置”，不会把代码越堆越硬。

  5) LLM 交互规范（降低一次性大生成的 bug）
  - 修复轮：强制返回“补丁格式”，只允许改动少量文件（例如 {patches:[{path,content,op}]}），避免模型每轮重写全仓导致漂移。
  - 观察输入：只喂“失败日志摘要 + 相关文件（有限数量/有限字节）+ 明确目标”，并设置 max_iters、timeout、stop_condition。

  6) 执行与安全（必须提前规划）
  - exec_tool 统一加：工作目录隔离、命令白名单/模板、CPU/时间/输出大小上限、端口占用检查、进程回收。
  - fs_tool 统一加：路径穿越防护、只允许写入 run 的 workspace。
  - 日志与密钥脱敏：event 里禁止落 API_KEY/AK/SK 原文。

  这个结构最小闭环：/generate/agent + RunStore(EventStore) + generate_validate 工作流（多轮 patch 修
  复），其余 workflow（上传 OSS、部署 FC）用同一套 step 机制往后叠加


# ***2026.3.26***

## 知识学习



## 实践

### Coding agnet的较复杂实现与调用：

**Coding Agent方案2：** ***/generate/agent***

方案2**函数调用流程图：**
  ```
  POST /generate/agent
  -> routers/agent.generate_with_agent()
  -> RunStore.create_run()
  -> run_generate_validate()
  -> build_enhanced_prompt()
  -> OpenAI-compatible /chat/completions 生成 files[]
  -> normalize_generated_files()
  -> materialize_files() 写入 run workspace
  -> _validate_workspace()
     -> prepare.sh
     -> build.sh
     -> start.sh / dev.sh
     -> HTTP probe
  -> 如果失败且 auto_fix=true
     -> pick_context_files()
     -> OpenAI-compatible /chat/completions 生成 patches[]
     -> apply_patches()
     -> normalize_generated_files()
     -> materialize_files()
     -> 再次验证
  -> RunStore.set_result() / set_status()
  -> GET /runs/{run_id} 轮询结果
```


方案2，根据大型agent框架构建的**数据流向：**
```
  1. 请求进入 generate_with_agent()。它把 prompt/template/model/max_iters/... 组装进 config，创建一个 run_id。见 Reference-framework/routers/
     agent.py:61、Reference-framework/routers/agent.py:78。
  2. RunStore.create_run() 会在本地创建目录：
     /tmp/reference-agent-runs/{run_id}/
     里面至少有：
  - run.json
  - events.jsonl
  - workspace/
    见 Reference-framework/agent/store.py:57、Reference-framework/agent/store.py:45。
  3. 如果 wait=false，路由只是 asyncio.create_task(coro) 把 workflow 丢到后台；如果 wait=true，当前请求会一直等到 workflow 结束。见 Reference-
     framework/routers/agent.py:95。
  4. workflow 入口是 run_generate_validate()。它先把 run 状态改成 running，然后调用 build_enhanced_prompt() 拼模板约束，再调用
     generate_project_with_openai()。见 Reference-framework/agent/workflows/generate_validate.py:230、Reference-framework/agent/workflows/
     generate_validate.py:252、Reference-framework/services/generation_rules.py:481。
  5. LLM 层当前不是 Responses API，也不是 Codex CLI 配置链路，而是后端自己读取 .env，直接请求 OPENAI_BASE_URL/chat/completions，并要求返回严格
     JSON schema。见 Reference-framework/services/openai_service.py:18、Reference-framework/services/openai_service.py:162、Reference-
     framework/services/openai_service.py:226。
  6. 生成结果被规范成 List[SaveFileItem]，也就是最核心的内部文件格式：
  - path
  - content
    见 Reference-framework/models.py:140。
  7. 生成完以后，会执行 normalize_generated_files()。这一步会强改模板相关内容，比如 Node18 依赖版本、补齐 scripts/*.sh、补 vite.config.js、修
     index.html 位置。见 Reference-framework/services/generation_rules.py:538。
  8. 然后 materialize_files() 把内存里的 files[] 写进 workspace/。注意这里是“覆盖同名文件”，不是“同步整个目录”。旧文件不会被删。见 Reference-
     framework/agent/tools/fs_tool.py:34。
  9. 如果 run_validation=false，workflow 到这里就结束，直接把 files[] 写入 run.result。见 Reference-framework/agent/workflows/
     generate_validate.py:264。
  10. 如果开启验证，会进入 _validate_workspace()：
  - 找一个空闲本地端口
  - 注入环境变量 WORKSPACE/HOST/PORT
  - 顺序跑 prepare.sh、build.sh
  - 再启动 start.sh，没有就退回 dev.sh
  - 用 HTTP 探测本地地址
    见 Reference-framework/agent/workflows/generate_validate.py:62、Reference-framework/agent/workflows/generate_validate.py:75、Reference-
    framework/agent/workflows/generate_validate.py:143。
  11. 验证时实际执行 shell 的地方在 exec_tool.py：
  - run_bash_script() 跑 prepare/build
  - start_bash_script() 起长跑进程
  - wait_http_ready() 轮询页面
  - 成功后马上 terminate_process()
    见 Reference-framework/agent/tools/exec_tool.py:37、Reference-framework/agent/tools/exec_tool.py:98、Reference-framework/agent/tools/
    exec_tool.py:191。
  12. 如果验证失败且 auto_fix=true，才会进入 patch 回路：
  - pick_context_files() 选少量关键文件
  - _build_fix_prompt() 拼失败日志和上下文
  - generate_project_patches_with_openai() 让模型返回 patches[]
  - apply_patches() 应用补丁
  - 再次 normalize_generated_files()
  - 再次 materialize_files()
  - 再验证
    见 Reference-framework/agent/workflows/generate_validate.py:323、Reference-framework/agent/tools/fs_tool.py:80、Reference-framework/
    services/openai_service.py:254。
  13. 当前 patch 的语义只有“按 path 覆盖/新增”，没有 delete，没有 rename，也没有“按最终 files 全量同步 workspace”。这正是你说要先修的 4。见
     Reference-framework/agent/tools/fs_tool.py:60。
  14. 成功或失败后，workflow 会把结果写回 run.json。result 里会直接带完整文件内容和 validation report。见 Reference-framework/agent/workflows/
     generate_validate.py:297、Reference-framework/agent/store.py:123。
  run 的状态和文件
  当前 run 的生命周期基本是：
  - queued
  - running
  - succeeded / failed / canceled
  状态变化和关键动作都会追加到 events.jsonl。见 Reference-framework/agent/store.py:119。
```

现在简化Agent流程：

**Coding Agnet 方案3：** ***/runs***

舍弃了大型agent的流程（）简化为简单的workflow。
**完整链路成功：**
  - 生成成功
  - 本机 prepare/build/start 全部成功
  - 发布成功
  - 已创建 OSS 项目和 FC
关键结果是：
  - run_id: run-1774496744666-42124bb9
  - project_id: proj-1774496798261
  - 最终状态：succeeded
  
简化成功debug成功。

### Docker打包成功：

(base) ryan@ENVYKatana:/mnt/c/Desktop/generate/Reference-framework$ curl --noproxy '*' -i http://127.0.0.1:9000/healthz
HTTP/1.1 200 OK
date: Thu, 26 Mar 2026 07:33:08 GMT
server: uvicorn
content-length: 166
content-type: application/json

{"ok":true,"service":"reference-framework","runs_dir":"/data/reference-agent-runs","checks":{"runs_dir_writable":true,"openai_configured":true,"oss_configured":true}}(base)



# ***2026.3.27***

## 知识学习

### 探讨Coding Agent架构

最终我们决定先完成强大的agent，再通过微调workflow的结构，格式化输出内容，以对接OSS/FC

而不是一开始根据OSS/FC的结构化需求设计Agent

逻辑关系是本末倒置的

### TypeScript

[TypeScipt教程](https://www.runoob.com/typescript/ts-tutorial.html)

非常多的开源项目用 TS 写成！


## 实践

### 设计能够格式化输出的Coding Agent

按现在这个目标，不需要把主流程改成“先输出 {"files":[...]} 再落盘”，具体而言：

现有 coding agent 的主链路：
1. 用户输入进入 SessionPrompt.prompt()
2. 主 agent 开始推理
3. agent 在过程中调用 write / edit / apply_patch
4. 文件直接写到当前工作目录，比如你指定的 TEST/

所以要加的，其实只是前置的一层：
1. 检测是否是“生成项目”意图
2. 如果不是，原样走现有聊天/分析流程
3. 如果是，先把简单 prompt 例如“生成贪吃蛇游戏”增强成一条完整、规范、工程化的 prompt
4. 然后把这条增强后的 prompt 继续交给现有 coding agent
5. 后面的文件创建仍然由现有 agent 正常完成

落实到改动操作：
1. IntentDetector
   输入简单 prompt，比如“生成贪吃蛇游戏”。
   输出结构化判断：intent=generate_project | normal_chat，以及 template=base-node18 | base-python39。
2. ProjectPromptEnhancer
   只在 intent=generate_project 时运行。
   用专门的 enhancer agent 深度理解、补全约束、规划输出。
   输出结构化结果建议是：
   template
   enhancedPrompt
   constraints
3. 把 enhancedPrompt 覆盖原始 text part，再把这条增强后的 user message 交给现有主生成链路。
   后面的文件生成仍然由 write/edit/apply_patch 完成，直接落到 TEST/




# ***2026.3.28***

## 知识学习

### Opencode

Opencode 内部调用逻辑


## 实践

### Coding Agent

我通过修改opencode的工作流，实现了我们项目所需的Coding Agent!

### 封装 Coding Agent 到 Fast API: 

我找到更稳的落点了：直接给 opencode 加一个正式 CLI 子命令，比如 generate-project --prompt ... 
这样 FastAPI 只需要调用这个命令拿 JSON，完全绕开“内部再起一个 HTTP 服务”的不稳定链路

我已经封装成功！
现在能稳定输出.json格式的文件。

### 删改opencode大项目结构

这很像我一年前删除Colias项目文件的过程......
已完成并存档。这是目前的baseline版本。

### 流式输出架构

我的流式推送接口应能：
1.我希望前端能展示出当前项目的文件结构文件树、已有的文件。
2.我希望不仅能够展示工作状态，还要有一行或者几行，实时显示LLM正在生成的代码。
3.我希望用“思考状态”展示agent的工作状态。


```推荐接口：

- POST /generate/jobs
  提交生成任务，返回 job_id
- GET /generate/jobs/{job_id}/stream
  SSE 流式输出
- GET /generate/jobs/{job_id}/result
  获取最终 { "files": [...] }
- GET /generate/jobs/{job_id}
  获取任务当前快照
- DELETE /generate/jobs/{job_id}
  可选，取消任务
```

为什么这样设计：
- 前端更容易接。POST 提交，GET 用 EventSource 收流。
- 能断线重连。
- 能同时拿“状态流”和“最终结果”。
- 不破坏现有 /generate。

```SSE 事件设计：
建议统一 JSON，事件名固定。
- job.created
  {"job_id":"gen_xxx","status":"queued"}
- thinking.status
  不传原始 CoT，只传安全摘要
  {
    "job_id":"gen_xxx",
    "phase":"planning",
    "label":"正在规划项目结构",
    "detail":"分析需求，确定模板和最小文件集"
  }
- project.tree
  前端文件树用这个
  {
    "job_id":"gen_xxx",
    "paths":["package.json","scripts/start.sh","src/main.js"]
  }
- file.snapshot
  当前已有文件内容
  {
    "job_id":"gen_xxx",
    "path":"src/main.js",
    "content":"..."
  }
- file.delta
  可选，给前端展示“刚写出来的几行代码”
  {
    "job_id":"gen_xxx",
    "path":"src/main.js",
    "append":"const app = ...\n"
  }
- preview.code
  专门给你第二个需求
  {
    "job_id":"gen_xxx",
    "path":"src/main.js",
    "lines":["const app = ...","app.listen(...)"]
  }
- session.status
  忙闲状态
  {
    "job_id":"gen_xxx",
    "status":{"type":"busy"}
  }
- job.completed
  {
    "job_id":"gen_xxx",
    "file_count":9,
    "result_url":"/generate/jobs/gen_xxx/result"
  }
- job.failed
  {
    "job_id":"gen_xxx",
    "error":"..."
  }
```

```怎么满足 3 个需求：

- 文件树、已有文件：
  用 project.tree + file.snapshot
- 实时显示 LLM 正在生成的代码：
  用 file.delta 或 preview.code
  这个不要依赖模型“口头输出代码”，而要监听真实文件写入
- 用“思考状态”展示 agent 状态：
  用 thinking.status内容来自阶段摘要，不直接暴露模型原始推理后端实现建议基于现有 packages/opencode/src/generate-api.ts 和 packages/opencode/src/server/generate-projectts改：
- Python FastAPI 负责：
    - 创建 job
    - 保存 job 状态
    - 返回 SSE StreamingResponse
- Bun 生成器负责：
    - 输出 JSONL 事件到 stdout
    - 最后一条输出最终 files
- 生成器内部增加 3 类 hook：
    - 阶段变化 hook：发 thinking.status
    - 文件变化 hook：发 project.tree、file.snapshot、preview.code
    - 结束 hook：发 job.completed
```

最重要的一点：thinking.status 不建议传模型原始 reasoning。
建议只传这种：
- 分析需求中
- 规划项目结构中
- 正在编写 package.json
- 正在修复启动脚本
- 正在校验最终结果
这样前端效果够好，也安全。


已实现

  改动在 app.py、packages/opencode/src/generate-api.ts、packages/opencode/src/server/generate-project.ts、packages/opencode/
  test/generate-api.test.ts。

  现在接口是：

  - POST /generate：保留原来的同步返回 { files: [...] }
  - POST /generate/jobs：创建流式生成任务
  - GET /generate/jobs/{job_id}：拿当前快照，里面有文件树、已有文件、预览代码、thinking 状态
  - GET /generate/jobs/{job_id}/stream：SSE 流
  - GET /generate/jobs/{job_id}/result：任务完成后拿最终 { files: [...] }

  流式行为现在支持：

  - 文件树：project.tree
  - 文件内容快照：file.snapshot
  - 代码预览：preview.code
  - 安全的工作状态：thinking.status
  - 任务状态：job.status
  - 最终结果：job.result

  另外，Bun 内部生成入口加了 --stream-jsonl，FastAPI 会把它解析成 job 状态并转成 SSE。同步 /generate 不受影响。reasoning 原文没
  有直接暴露给前端，只保留了状态级事件。


# ***2026.3.29***

## 毕设毕设！





# ***2026.3.30***

## 知识学习

### 对现有工程的思考：我们的目的是什么？

***真正缺的，不是“再造一个能连续对话的 /generate”，而是把 /generate 里更强的那部分能力搬进 web 的标准 session 路径里***

#### 如果是workshop，那重心在于强大的coding agent、在线运行。但“生成自己的openclaw”这样级别的要求难以实现  
“贪吃蛇”是远远不够的。目前，coding agent已经足够强大，但在云端运行“用户生成的程序”仍需细致对接  


#### 如果是能解决具体需求的agent，那重心在于SKILL.md, 理解需求、细致的构造pipeline，通过多个skill真正去解决问题

##### TODO List: 

- find skills的SKILL.md, 进一步的，一堆工具skills，如何调用

- 需求分析的SKILL.md

- 不同应用场景，写paper，写report，上网

- AI降重：带上自己的思想;  降重 SKILL.md

- 让claude触手可得



## 实践

### Coding Agent网页版在线运行

在现有网页应用上加OSS/FC在线运行：

#### 第一阶段
```
  - 从 packages/opencode/src/server/generate-project.ts 抽出“项目 contract 校验”模块。
  - 保留现有规则：模板识别、必需文件、scripts/*.sh 约束、Vite/Vue 版本约束、Python/FastAPI约束。
  - 目标文件建议新建为 packages/opencode/src/server/project-contract.ts 或相近位置。
  - /generate 和 web session 后续都复用这一个模块。
```
#### 第二阶段
```
  - 在 web 端加“在线构建并在线运行模式”开关。
  - 入口优先放在 packages/app/src/components/prompt-input/submit.ts 所在提交链路附近。
  - 这一步只做“把模式标记传到后端 prompt 流程”，不要先做部署。
```
#### 第三阶段
```
  - 扩展 packages/opencode/src/plugin/prompt-enhancer.ts，让它在该模式下启用更强 project contract。
  - 普通聊天不受影响。
  - 项目生成首条消息启用：固定模板规则、输出目录规则、运行脚本规则、可部署规则。
```
#### 第四阶段
```
  - 在标准 session 流里接入“校验失败自动修复”。
  - 不再走临时 /generate session。
  - 在同一 session 内，首轮生成完成后校验；失败时自动追加 repair prompt；成功则结束。
  - 这样 web 的持续对话、流式状态、消息历史都天然保留。
```
#### 第五阶段
```
  - 只约束部署必需接口，不先绑定具体云厂商实现细节。
  - 先稳定：
    scripts/prepare.sh
    scripts/build.sh
    scripts/start.sh
    HOST
    PORT
    WORKSPACE
```

保留 /generate: 作为外部 API、作为 contract 回归测试入口、作为未来批量生成入口

最值得先做的是第 1 步和第 2 步。做完这两步，主路径就从“独立 /generate”转成“web session + 强约束项目模式”。
下一步直接改两处：packages/opencode/src/server/generate-project.ts、packages/opencode/src/plugin/、prompt-enhancer.ts，然后再接 web 的模式开关。

现在 web 版已经接上这条链路了： 
新会话输入框里新增了一个持久化的 “Run Online / 在线构建运行” 开关  
打开后，首条普通 prompt 会带上在线运行 mode marker；后端prompt-enhancer会据此启用之前加的“在线构建并运行”约束  
相关改动在 packages/app/src/components/prompt-input.tsx  
packages/app/src/components/prompt-input/submit.ts  
packages/opencode/src/plugin/prompt-enhancer.ts  
以及新的共享模块 packages/util/src/project-mode.ts  




# ***2026.3.30***

## 知识学习

### Workshop数据流动、架构细节分析讨论

### Seminar: World Model 


## 实践

### Debug Web应用

已添加“构建项目”模式，并在前端展示现已生生成的文件

带前端版本的完整pipeline已跑通！

现在，对接CUHKSZ的第二前端。





# ***2026.3.31***

## 知识学习




## 实践





# ***2026.4.1***

## 知识学习

### Claude Code 框架

Claude Code 源码泄露...

学一波：[Bilibili: Claude Code源码泄露！首发解读51万行代码！](https://www.bilibili.com/video/BV11zXCBFEMo/?spm_id_from=333.1387.top_right_bar_window_default_collection.content.click)


## 实践

## [Workshop 架构重构：MVP 版本](https://github.com/Ryannnice/Agent-Do)

我们非常简洁、高效地实现了workshop。

效果很好，并且摒弃了opencode的庞大源代码植入。

优化前端，现在反应简介迅速美观。




# ***2026.4.2***

## 知识学习

### Docker 

[Bilibili Docker](https://www.bilibili.com/video/BV1THKyzBER6/?spm_id_from=333.1387.favlist.content.click&vd_source=487ef5084994b81a0ec05eeffa991ed2)

### 关于项目未来的讨论

教育平台是核心竞争力

下一阶段定位：用户每天早晨五分钟了解AI圈大事，甚至上手实践，学习一二。


## 实践

## Workshop 架构重构：MVP 版本

Debug, 增添流式输出功能


### Claude Code 宠物系统修复

已经成功部署运行[“开源”的Claude Code](https://github.com/Ryannnice/claude-code)!

修复了桌面宠物 /buddy 功能！


### Git Branch

常见 type（非常重要）
```
feature	  新功能	  feature/payment-api
fix   	  修复 bug  fix/order-null-pointer
hotfix  	紧急修复	hotfix/login-crash
refactor	代码重构	refactor/cache-module
docs	    文档修改	docs/api-guide
test	    测试代码	test/user-service
chore	    杂项修改	chore/dependency-update
```


### Workshop

尝试构建多项目运行时，并展示：

我们发现似乎并不需要FC或者公网URL。





# ***2026.4.3***

## 知识学习

### git / docker

git pull / docker compose up --build -d dasd

› 我现在要学习第一部分了。

  我的基础很薄弱，尤其是deep learnning，只学过理论没实战过。
  我自学过一遍Transformer，但记不住太多。

  现在，你的任务是帮我彻底搞懂第一部分。
  我们现在开始
───────────────────────────────────────────────────────────────────────────────────────────────────────────────

• 起点

  我们先把“第一部分”理解成 assignment1-basics。
  它不是单独讲 Transformer，而是在做一件完整的事：

  把原始文本变成 token，用 Transformer 预测“下一个 token”，再用 loss 训练它。

  这部分你只要先抓住一条主线：

  文本 -> tokenizer -> token id -> embedding -> Transformer -> logits -> cross entropy loss -> 反向传播 -> 参数
  更新

  这条线一旦明白，后面每个文件都会变得很顺。

  你要先记住的 8 个概念

  1. token
     文本不会直接喂给模型，先切成更小单位。这里用 BPE tokenizer。
  2. token id
     每个 token 都映射成一个整数，比如 314、9821。
     整数不能直接做神经网络计算，所以先查表变成向量。
  3. logits
     模型最后输出一个长度等于词表大小的分数向量。还不是概率，只是“每个 token 有多像下一个答案”。
  4. target
     训练目标是“下一个 token 是什么”。
  5. loss
     如果模型给正确 token 的分数不够高，就会有损失。
  6. parameter
     模型里所有可学习的权重，比如线性层权重、embedding 表、norm 权重。
  7. 训练
     用 loss 的梯度更新 parameter，让模型下次预测更准。

  这个作业到底在实现什么

  按代码分，assignment1-basics 主要有 5 块：

  1. Tokenizer: assignment1-basics/cs336_basics/tokenizer.py
  2. 基础神经网络模块: assignment1-basics/cs336_basics/model/modules.py
  3. Transformer 组装: assignment1-basics/cs336_basics/model/transformer.py
  4. 数据采样与 loss: assignment1-basics/cs336_basics/trainer/data_loading.py, assignment1-basics/cs336_basics/
     trainer/utils.py
  5. 训练脚本: assignment1-basics/cs336_basics/train.py

  先看全局流程
  假设原文是：

  I love deep learning

  训练时不会直接让模型输出整句话，而是做“下一个词预测”。

  如果 token 化后得到：

  [10, 25, 88, 91]

  那么一次训练样本会变成：

  输入 x: [10, 25, 88]
  目标 y: [25, 88, 91]

  意思是：
  - 看到 10，预测下一个是 25
  - 看到 10, 25，预测下一个是 88
  - 看到 10, 25, 88，预测下一个是 91

  这就是 language modeling。

  这件事在 assignment1-basics/cs336_basics/trainer/data_loading.py 里很直白：它从长 token 序列里随机切一段作为
  inputs，再整体右移一位作为 targets。

  Transformer 在这里扮演什么角色

  模型本体在 assignment1-basics/cs336_basics/model/transformer.py。

  它做的事可以概括成：

  1. 用 Embedding 把 token id 变成向量
  2. 经过多个 transformer_block
  3. 做一次 RMSNorm
  4. 用最后一个线性层投影到词表大小
  5. 得到每个位置的 logits

  也就是：

  token ids -> token vectors -> blocks -> normalized vectors -> vocab logits

  你现在不用死记结构，先记一句：

  Transformer 的工作，就是把“上下文向量”一步步加工成“下一个 token 的打分”。

  一个 block 里发生了什么

  在 assignment1-basics/cs336_basics/model/transformer.py 的 transformer_block 里，核心是：

  x = x + self.attn(self.ln1(x))
  x = x + self.ffn(self.ln2(x))

  这是标准的 Pre-Norm 结构。
  你可以这样理解：
  1. 先归一化
  2. 做注意力，让每个位置“看”上下文
  3. 残差加回原输入
  4. 再归一化
  5. 做前馈网络，提升表达能力
  6. 再残差加回去

  这里最重要的不是公式，而是功能分工：

  - Attention 负责“从别的位置拿信息”
  - FFN 负责“对当前位置的信息做非线性加工”

  基础模块都是什么

  看 assignment1-basics/cs336_basics/model/modules.py：

  - Linear
    最基础的线性变换，本质是矩阵乘法。
  - Embedding
    本质是查表：输入 id，输出向量。
  - RMSNorm
    做归一化，稳定训练。
  - SwiGLU
    是前馈层的非线性结构，比普通 ReLU 更现代。
  - scaled_dot_product_attention
    真正算注意力分数和加权求和的地方。
  - multihead_self_attention
    把注意力拆成多个 head 并并行计算。
  - RotaryPositionalEmbedding
    给 Q/K 注入位置信息，也就是 RoPE。

  你最该先懂的数学只有两块

  第一块：线性层
  你可以把它看成：

  输出 = 输入 × 权重矩阵

  第二块：softmax + cross entropy
  在 assignment1-basics/cs336_basics/trainer/utils.py 里，cross_entropy 做的是：

  - 取出正确答案那个 logit
  - 和所有 token 的总体竞争结果 logsumexp 比较
  - 正确答案分数越高，loss 越小

  你现在只要记住：

  loss 小 = 模型更相信正确答案

  为什么 attention 重要

  如果没有 attention，每个位置只能“各算各的”。
  有了 self-attention，当前位置可以看前面所有位置。

  比如句子：

  The animal didn't cross the street because it was too tired.

  模型在处理 it 后面的词时，attention 可以帮助它回头看 animal，知道这里的 it 更像指动物。

  在这个作业里，attention 的核心实现在 assignment1-basics/cs336_basics/model/modules.py。

  你先只记这个流程：

  1. 输入向量投影成 Q, K, V
  2. 用 Q·K 算相关性
  3. 用 softmax 变成权重
  4. 用权重加权求和 V

  为什么还需要 RoPE

  Transformer 本身如果只看 token 向量，是不知道先后顺序的。
  RoPE 的作用，就是把“位置”编码进 Q 和 K，让 attention 知道“谁在前，谁在后”。

  你不需要一开始就吃透公式。先记一句：

  RoPE 不是给 token 加一个位置向量，而是直接旋转 Q/K，让相对位置信息进入注意力分数。
  3. 再看 assignment1-basics/cs336_basics/model/modules.py
     一个一个理解基础模块。
  4. 再看 assignment1-basics/cs336_basics/model/transformer.py
     看这些模块如何拼成整个模型。
  5. 最后看 assignment1-basics/cs336_basics/train.py
     把前向、loss、优化器、checkpoint 串起来。

  我们接下来怎么学

  我建议分 6 步带你彻底搞懂：

  1. 先彻底搞懂“输入、目标、loss”这条线
  4. 再搞懂 Transformer block
  5. 再搞懂 tokenizer / BPE
  6. 最后把训练脚本完整跑通





› 好的。请继续带我走，配合代码讲解。
  最重要的是，你需要在每一句代码上方【都】加上中文注释！！！（从我们目前讲
  解的部分开始）

• 已经继续好了。当前这两个文件已经变成逐行中文注释版：

  - assignment1-basics/cs336_basics/trainer/data_loading.py
  - assignment1-basics/cs336_basics/trainer/utils.py

  现在我们正式讲第一讲：

  第一讲
  输入、目标、loss 到底是什么

  先只看 assignment1-basics/cs336_basics/trainer/data_loading.py。

  它做的事情非常朴素：

  从一长串 token 里，随机切一段出来当输入；
  再把这段输入整体右移一位，作为监督目标。

  比如数据集里有一串 token：

  [10, 25, 88, 91, 7, 3, 42]

  如果某次随机采样从 start_idx = 1 开始，context_length = 4，那么：

  input_seq    = [25, 88, 91, 7]
  input_target = [88, 91, 7, 3]

  意思是：

  - 看到 25，预测下一个是 88
  - 看到 25, 88，预测下一个是 91
  - 看到 25, 88, 91，预测下一个是 7
  - 看到 25, 88, 91, 7，预测下一个是 3

  这就是语言模型训练的本质。

  你必须真正吃透的一点

  模型从来不是“直接学一句话的意思”。

  它学的是：

  给定前面的 token，预测下一个 token。

  这是你后面看所有 Transformer 代码的总钥匙。

  把 data_loading.py 翻译成人话

  在 assignment1-basics/cs336_basics/trainer/data_loading.py 里：

  - dataset_len = len(dataset)
    先知道整个 token 数据有多长。
  - inputs = torch.empty(...)
    先准备好一个输入 batch 的壳子。
  - targets = torch.empty(...)
    再准备好一个目标 batch 的壳子。
  - for i in range(batch_size):
    一条一条构造样本。
  - start_idx = ...
    随机选起点。
  - input_seq = dataset[start_idx : start_idx + context_length]
    取输入片段。
  - input_target = dataset[start_idx + 1 : start_idx + context_length + 1]
    取右移一位后的目标片段。

  最后得到两个张量：

  - inputs.shape = (batch_size, context_length)
  - targets.shape = (batch_size, context_length)

  例如：
  inputs  =
  [[25, 88, 91,  7],
   [13,  5, 99,  2]]  
  targets =
  [[88, 91,  7,  3],
   [ 5, 99,  2, 17]]  

  接下来是 loss

  看 assignment1-basics/cs336_basics/trainer/utils.py 里的 cross_entropy()。

  先别怕这个名字，它其实只是在做一件事：

  检查模型有没有把正确答案打高分。

  假设某个位置，模型输出的 logits 是：

  [1.2, 0.1, 3.5, -0.7]

  假设正确答案 token id 是：2  
  因为第 2 号位置对应分数 3.5，说明模型对正确答案比较自信，那么 loss 就会比
  较小。

  如果正确答案是：1  
  那正确答案分数只有 0.1，loss 就会比较大。

  所以 cross_entropy 你可以先理解成：

  正确答案分越高，loss 越小；正确答案分越低，loss 越大。

  这个函数内部到底在算什么

  在 assignment1-basics/cs336_basics/trainer/utils.py 里有三步： 
  1. gather(...)
     从整行 logits 里，取出“正确 token 对应的那个分数”。
  2. logsumexp(...)
     计算所有候选 token 的总体竞争强度。
     你可以先把它理解成 softmax 分母的对数稳定版本。
  3. loss = -get_logit + logsumexp
     正确答案分数越大，这一项越小。

  这就是交叉熵。
  一个最小训练样本，完整走一遍  
  下面这个小例子你要反复看，直到脑子里有画面。

   假设一条输入序列长度是 4。
  x = [25, 88, 91, 7]

   对应目标序列也是长度 4。
  y = [88, 91, 7, 3]

   模型会对 x 的每个位置都输出一个“整个词表大小”的分数向量。
  logits.shape = (4, vocab_size)

   第 0 个位置，模型根据 25 去预测 88。
  logits[0] -> 预测目标 y[0] = 88

   第 1 个位置，模型根据 25,88 去预测 91。
  logits[1] -> 预测目标 y[1] = 91

   第 2 个位置，模型根据 25,88,91 去预测 7。
  logits[2] -> 预测目标 y[2] = 7

   第 3 个位置，模型根据 25,88,91,7 去预测 3。
  logits[3] -> 预测目标 y[3] = 3

  然后：

  - 每个位置算一个 loss
  - 所有位置平均
  - 得到这次 forward 的总 loss
  - 再 backward 更新参数

  你现在要建立的第一个直觉

  训练时，模型不是一次只预测最后一个 token。
  它会对序列里的每个位置都做“下一个 token 预测”。

  所以：

  - 输入一段长度为 T 的 token
  - 输出其实是 T 组 logits
  - 每个位置都能贡献训练信号

  这就是为什么训练效率高。

  现在把这件事和 Transformer 连起来

  你可以把当前第一部分先简化成一句话：

  - data_loading.py 负责造题目和标准答案
  - transformer.py 负责根据题目做预测
  - utils.py 负责给预测打分
  - 优化器负责按这个分数去修正模型

  也就是：

  造训练样本 -> 模型预测 -> 计算损失 -> 更新参数

  你此刻应该记住的 3 句话

  1. inputs 是当前看到的 token 序列。
  2. targets 是右移一位后的“下一个 token”答案。
  3. cross_entropy 是在检查模型有没有把正确 token 打高分。

  下一步怎么走

  下一讲我建议直接进入：

  Embedding 和 Linear 到底是什么

  因为它们是 Transformer 最基础的积木。
  如果这两个你真的懂了，后面的 attention 会轻松很多。

  如果你同意，我下一条就继续讲第二讲，并且从：

  - assignment1-basics/cs336_basics/model/modules.py
    里的 Embedding
  - Linear



## 实践

### Docker构建Claude Code

遇到构建docker镜像时候地址错误问题：

- Claude 看起来执行了
- 但改动写进了错误挂载目标
- 当前 session 的真实 workspace/ 还是空的
- 然后 runtime/start 检测不到 index.html / package.json，就返回 400 当前 session 没有可在线运行的项目
  
问题不是 HTTP 400 本身，而是 Agent-Do 之前把 Claude 子容器的 workspace 挂到了错误的宿主机路径。
AGENT_DATA_HOST_ROOT 在 Backend/WorkShop/.env 里指向了旧机器上的 /root/internship-szdsjyjy/...，
但这台机器真实路径是 /home/ryan/CUHKSZ/Education_Platform/Backend/Agent-Do/data。

- 修正了 Agent-Do 容器内 Docker CLI 的集成，避免之前的 input/output error: 'docker'
- 修正了 Backend/WorkShop/.env 里的 AGENT_DATA_HOST_ROOT，并重启了 agent-do


当前 qwen3-coder-next 这条模型链路有时会“看起来成功，实际上没写任何文件”。
我准备从 WorkShop 侧加兜底：当第一轮生成后没有形成可预览项目时，自动发一轮更强约束的修复 prompt，而不是直接把 runtime/start 400 暴露给前端。




# ***2026.4.4***  
## 毕设毕设！  
## 知识学习  
### PageAttention   
[怎么加快大模型推理？10分钟学懂VLLM内部原理，KV Cache，PageAttention](https://www.bilibili.com/video/BV1kx4y1x7bu/?spm_id_from=333.1391.0.0&vd_source=487ef5084994b81a0ec05eeffa991ed2)
### Flash Attention   
[Flash Attention 为什么那么快？原理讲解](https://www.bilibili.com/video/BV1UT421k7rA/?spm_id_from=333.1391.0.0&vd_source=487ef5084994b81a0ec05eeffa991ed2)

# ***2026.4.6***  
## 拯救计划很好看    
我觉得可以和星际穿越媲美。  
太空的浪漫很纯粹。  
[Bilibili细节解析](https://www.bilibili.com/video/BV1oSQZBRE8j/?spm_id_from=333.337.search-card.all.click)  

# ***2026.4.5***  
## 毕设毕设！  




# ***2026.4.7***

## 毕设毕设！

## 知识学习

### AI Station 

[AI Station 教程](https://xxl9u0uq9y2.feishu.cn/wiki/LVHvw3GCWiMlV4kjH25clngHnVf)

### LLM Router

谕书的完整Pipeline

### Router 综述

#### LLM 路由的概念设计空间
本综述涵盖的范式（参见 1.3 节）为组织和理解文献提供了基础 。
在实践中，现实世界的系统往往同时借鉴了不止一种范式 。
为了补充基于范式的组织方式，路由方法还可以从更广泛的维度进行分类 ：

##### 决策时机 (When)：指路由决策何时做出 。
路由系统可以依赖生成前 (Pre-generation) 决策或生成后 (Post-generation) 决策，也可以采用多阶段过程 。
生成前路由在产生任何输出前选择模型，完全依赖于输入查询的属性；而生成后路由则在产生初始响应后，根据输出质量或置信度信号做出决定 。

##### 使用信息 (What)：路由机制使用的信号丰富程度各不相同 。最简单的方法仅基于查询本身，利用词法或语义特征来刻画请求 。
更进阶的系统还会加入可用模型的元数据来指导选择，如成本、延迟或领域专长 。生成后方法则进一步引入响应级信号，如置信度得分、Token 概率或验证器输出 。

##### 计算方式 (How)：路由决策的计算复杂度差异显著 。一端是简单的阈值规则或基于成本的启发式方法，无需训练即可直接应用 ；
另一端是基于历史表现数据训练的监督分类器，用于预测哪个模型最适合处理给定查询 。
更复杂的方法采用自适应策略，通过与环境的持续交互来更新路由行为 。


#### 主流技术路线
1. 难度感知路由 (Difficulty-aware Routing)
  这是最直观的路线，核心是**“看题下菜”** 。
  原理：在推理前评估查询的复杂度，将简单题分给小模型，难题分给大模型 。
  评估手段：包括启发式规则（如文本长度、词汇稀缺度）、训练专门的分类器（如你计划使用的 0.5B 模型）或使用 “LLM 作为评判者” 。
  代表案例：BEST-Route（动态分配并选择采样策略）和 VLLM Semantic Router（识别是否需要开启昂贵的思维链推理） 。
2. 人类偏好对齐路由 (Human Preference-aligned Routing)
  不看“对错”，看**“好坏”** 。
  原理：模拟人类的主观评价，预测大模型生成的答案是否会比小模型显著“更好” 。
  训练数据：利用 Chatbot Arena 等人类真实偏好数据或 LLM 自动生成的对比数据 。
  代表案例：RouteLLM（预测强模型是否会胜出）和 Arch-Router（允许用户自定义不同领域的路由偏好） 。
3. 基于聚类的路由 (Clustering-based Routing)
  核心是**“找规律”** 。
  原理：利用无监督学习（如 K-means）将语义相似的查询聚类，并为每个簇分配表现最好的模型 。
  优势：具有极强的扩展性，添加新模型时无需重新训练路由器，只需测试新模型在各个簇上的表现即可 。
  代表案例：UniRoute 和 Avengers-Pro 。
4. 强化学习路由 (Reinforcement Learning Routing)
  核心是**“实战进化”** 。
  策略优化：通过多步交互（思考 -> 路由 -> 再思考）迭代改进答案，适合复杂推理，但延迟较高（如 Router-R1） 。
  在线老虎机 (Bandit)：在实时交互中通过用户反馈（点赞/踩）动态调整路由策略，平衡“探索新模型”与“利用已知强模型” 。
  代表案例：MixLLM（实现 97% 的 GPT-4 质量且仅需 24% 的成本） 。
5. 基于不确定性的路由 (Uncertainty-based Routing)这是你项目中 Logprobs 熵值 策略的理论依据 。
  原理：监控模型对自身回答的“信心” 。如果内部数学信号（如概率分布）显示模型在犹豫，则触发升级 。
  关键点：研究证明，模型内部的探测信号（Logits）远比模型自己口头说的“我很确定”要准得多 。
  代表案例：CP-Router（利用共形预测处理不确定性） 。
6. 级联系统 (Cascading)这是你项目中 Binary Gate 和逐级踢球架构的归属 。
  原理：顺序执行。先让小模型试，不行再给中模型，最后大模型保底 。
  核心逻辑：引入了“后悔药”机制，通过自我验证或外部评估器决定是否停止或升级 。
  代表案例：FrugalGPT（三大组件：路由器、质量评估器、停止判断器）和 AutoMix 。


# ***2026.4.8***

## 实践

搭建了自己的完整Pipeline
质量不变的情况下，cluster方法效果最好，成本降低了 ***10%***

## 知识学习

### Router 经典论文总结
本文档面向后续逐篇复现，聚焦综述 《Dynamic Model Routing and Cascading for Efficient LLM Inference: A Survey》 中以下三节的代表性工作：

Section 2: Difficulty-aware Routing
Section 6: Uncertainty-based Routing
Section 7: Cascades
整理原则：

只优先采用原论文、官方项目页、官方代码仓库、会议页面。
如果某些细节在摘要页看不到，我会明确标注“需要补读 PDF/附录”。
如果仓库 README 展示的是论文发布后的扩展结果，我会明确写成“仓库后续更新”，避免和论文主结果混淆。
一页结论
如果你接下来要逐一复现，我建议按这个顺序推进：

AutoMix：代码、数据、任务说明最完整，最适合先跑通一个 cascade 基线。
FrugalGPT：工程可用性强，官方仓库完整，适合改造成商业 API 版本。
BEST-Route：代码完整，但包含 reward model、best-of-n、多阶段数据构造，工程复杂度高于前两者。
GraphRouter：官方代码已放出，但图构建与数据预处理更复杂。
EmbedLLM：数据和代码齐全，但更像“模型表示学习 + routing 下游头”，对实验环境要求更高。
CP-Router：训练自由、思路清晰，但我当前未检索到官方代码，复现需要自己补实现。
Self-REF / Learning to Route LLMs with Confidence Tokens：论文价值高，但目前未检索到官方公开代码。
Confidence-Driven LLM Router：适合后续用商业 API 重做，但目前主要能拿到论文页面信息，代码未公开。


### 开源Router方案总结
本文档对 4 个你点名的开源 router / router 模型做统一拆解：

1. `RouteLLM`
2. `semantic-router`
3. `notdiamond-0001`
4. `knn-router`  
整理维度尽量与 `经典论文.md` 保持一致： 
- 项目定位
- 相关论文或技术来源
- 数据集
- 测试用大模型 / 候选模型池
- router 模型 / 机制
- 效果 / benchmark
- 创新点
- 实验与工程形态
- 开源代码位置
- 复现建议


### RouteLLM

最后这处 I/O 问题已经补上了。你现在不需要再改代码了，直接按下面的命令走就行。

• 现在 GSM8K 这条链路已经打通到最后一步了。
  刚才 2 题 smoke test 的实质结果是：

  - router-random-1.0 已经成功生成 weak model 结果
  - router-random-0.0 已经成功生成 strong model 结果
  - 最后失败只是因为 outputs/ 目录不存在
  - 我已经把脚本改成自动创建输出目录了

  也就是说，你现在可以直接正式跑。

  你刚才那条命令的正确写法
  你那次输入里混进了重复命令、\\ 和 \del。后面请直接复制下面这条，不要自己改换行符：

  cd ~/CUHKSZ/LLM-Router/RouteLLM
  source .venv/bin/activate

  .venv/bin/python -m routellm.evals.gsm8k.generate_responses \
    --host http://127.0.0.1 \
    --port 6060 \
    --parallel 4 \
    --limit 5 \
    --strong-model openrouter/qwen/qwen-2.5-72b-instruct \
    --weak-model openrouter/qwen/qwen-2.5-7b-instruct \
    --output-file outputs/gsm8k_qwen_smoke.csv

  如果 smoke test 成功，再跑全量

  .venv/bin/python -m routellm.evals.gsm8k.generate_responses \
    --host http://127.0.0.1 \
    --port 6060 \
    --parallel 8 \
    --strong-model openrouter/qwen/qwen-2.5-72b-instruct \
    --weak-model openrouter/qwen/qwen-2.5-7b-instruct \
    --output-file outputs/gsm8k_qwen72b_7b.csv

  我建议先用 --parallel 8，别一开始开太大，OpenRouter 更稳。

  然后做评测和可视化

  .venv/bin/python -m routellm.evals.evaluate \
    --routers random bert \
    --benchmark gsm8k \
    --strong-model openrouter/qwen/qwen-2.5-72b-instruct \
    --weak-model openrouter/qwen/qwen-2.5-7b-instruct \
    --gsm8k-responses outputs/gsm8k_qwen72b_7b.csv \
    --output outputs/gsm8k_qwen72b_7b_eval \
    --num-results 10 \
    --parallel 4 \
  - 图：outputs/gsm8k_qwen72b_7b_eval/gsm8k.png
  - cache：outputs/gsm8k_qwen72b_7b.cache.npy

  你现在实际应该怎么做

  1. 保持 routellm.openai_server 还在运行
  2. 先执行 5 题 smoke test
  3. 如果成功，再执行全量生成
  4. 最后执行 evaluate 出图

  这次我修掉的关键问题

  - bert 路径不再强依赖 OPENAI_API_KEY
  - openai_server.py 不再在 import 阶段崩
  - gsm8k.generate_responses 支持自定义模型对和输出文件
  - gsm8k.generate_responses 修复了过时的 OpenAI backend 构造，改为正确使用 sglang 的 backend
  - 输出目录现在会自动创建
  - evaluate.py 支持读取你自己的 GSM8K 响应 CSV 并直接可视化

  如果你跑完 smoke test，把终端输出贴给我。我可以直接帮你判断：

  - 可以上全量
  - 还是还要调 parallel
  - 或者要不要顺手把 MMLU 也一起改成同样流程




# ***2026.4.9***

## 尝试更多的开源策略：

### [RouteLLM](https://github.com/aurelio-labs/semantic-router)

准确率（Accuracy）:

在 GSM8K 数据集上，不同策略的表现如下：
策略              准确率 (Accuracy)    相比 Random 的提升                     评价
Random (随机)        88.93%                  -                     基准线：无脑混合强弱模型的结果。
Causal_LLM         0.52%+1.59%           显著胜出：                成功识别了模型专长，捕获了互补性 。
MF (矩阵分解)      90.30%+1.37%           优于随机：                即使只有部分数据，也展现了预判能力。
BERT/SW_Ranking     ~88.7%             -0.2%(负优化)           低于随机：说明这些路由器在数学逻辑上出现了误判。


策略,               准确率 (Accuracy),   成本 (CNY),性能/成本效率评价
Weak (7B),          85.90%,0.58,        成本极低，但存在能力天花板
Strong (72B),       92.87%,1.77,        准确率最高，但成本是 7B 的 3.06 倍
Causal_LLM (Router),90.52%,1.20,        最优解：用 67% 的成本换取了 97.5% 的最强性能


# ***2026.4.10***


## 尝试更多的开源策略：

### 现有Pipeline

现有 llmrouter 逻辑
  整体主流程在 llmrouter/scripts/run_all.sh:26：

  数据合并与切分
  -> 全模型 benchmark
  -> 自动打 tier 标签
  -> 训练 classifier + 调 cascade 阈值
  -> 在 test 上统一评测所有路由策略
  -> 出图

  核心分层是：

  - 数据池构建： llmrouter/data/merge_and_split.py:45
    把 ceval/cmmlu/gsm8k 和 mixed_test_full 合并，按 dataset 做 80/20 stratified split，输出 data/unified/
    train.jsonl 和 data/unified/test.jsonl。这里是整个 pipeline 的 train/test 边界。
  - Prompt 构造与答案判定： llmrouter/src/benchmark/datasets.py:73
    build_prompt 负责把原始 query 变成给模型的 prompt；check_answer 负责按 MCQ/数学题规则判分。这决定了 benchmark
    的 correct 字段。
  - 全模型 benchmark： llmrouter/src/benchmark/run_benchmark.py:55
    通过 OpenAI SDK 去调用本地 vLLM 服务。每个 query 对每个模型都会真实调用一次，保存：
    id/query/prompt/output/correct/latency_ms/logprobs/...
  - 自动标签： llmrouter/src/labeling/auto_label.py:59
    从 benchmark 结果里找“最小能答对的模型 tier”，生成 routing_label。如果所有模型都错，仍记为 label=4，并打
    all_wrong=True。
  - Classifier 训练： llmrouter/src/router/train_classifier.py:224
    用 routing_labels.jsonl + unified/train.jsonl 训练一个 Qwen2.5-0.5B 5 分类器。输入是 query，前面再拼 1.5B 的
    logprob 置信 token；训练时过滤 all_wrong 样本。训练完还会顺手在训练集上调优 cascade 阈值。
  - 统一评测： llmrouter/src/evaluate/run_evaluation.py:385
    这一步不是在线真实路由，而是：
      1. 载入 test split 的 benchmark 结果
      2. 给 test 重新生成 oracle label
      3. 运行 classifier 推理
      4. 对每种策略做“模拟路由”
  - 指标统计： llmrouter/src/benchmark/metrics.py:100
    统一算 accuracy、分数据集/难度准确率、latency、routing distribution、cost ratio。

  模型调用与测评方式
  这里最关键的是分清“真实调用”和“离线模拟”。

  - 真正调用模型只发生在 benchmark 阶段，见 llmrouter/src/benchmark/run_benchmark.py:62。
    它用 OpenAI-compatible API 调 vLLM，温度固定 0.0，并拉取 logprobs/top_logprobs。
  - 训练 classifier 时，不再调用大模型回答问题，只加载一个 0.5B 分类模型做监督训练，见 llmrouter/src/router/
    train_classifier.py:246。
  - 评测阶段也不再真正调用被选中的 Qwen。
    simulate_strategy 只是根据路由结果选一个 tier，然后去 benchmark 表里查这个模型在该 query 上的既有结果，见
    llmrouter/src/evaluate/run_evaluation.py:269。
  - 这意味着当前论文/实验口径里：
      - accuracy 是公平可比的
      - cost ratio 也是公平可比的
      - latency 只统计“被路由到的模型推理延迟”或 cascade 累积延迟
      - flat router 自己的 routing latency 其实没被计入；classifier 和未来 semantic-router 都是这样

  现有策略集合在 llmrouter/src/evaluate/run_evaluation.py:433：

  - baseline-32b / 14b / 7b / 3b / 1.5b
  - random
  - oracle
  - cascade-default
  - cascade-optimized
  - classifier
  - binary-gate-logprobs

  另外，仓库里还有在线 runtime router，但它们不属于当前论文评测主链：
  - llmrouter/src/router/classifier.py:25
  - llmrouter/src/router/cascade.py:82


### [Semantic-router](https://github.com/aurelio-labs/semantic-router)
  semantic-router 结构
  你要接的源码现在已经 vendoring 到 llmrouter/semantic_router，原始上游目录没动，说明见 llmrouter/semantic_router/
  VENDORED.md:1。
  核心结构：
  - Route：定义 route name 和一批 utterances，见 llmrouter/semantic_router/route.py:48
  - SemanticRouter：dense embedding 路由器，见 llmrouter/semantic_router/routers/semantic.py:15
  - BaseRouter.__call__：真实调用入口，见 llmrouter/semantic_router/routers/base.py:571
  - route scoring：top-k utterance 检索后按 route 聚合打分，见 llmrouter/semantic_router/routers/base.py:1536

  调用流是：
  定义 Route(name, utterances)
  -> encoder 把 utterances 向量化
  -> index 建库
  -> query 编码
  -> top-k 检索相似 utterances
  -> 按 route 聚合分数
  -> 返回 RouteChoice(name, similarity_score)

  所以它本质上是“检索式分类器”，不是 LoRA classifier，也不是偏好模型。
  如何接入 llmrouter
  最合理的接入点是 Phase 3 evaluation，作为一个新策略 semantic-router。不要改 benchmark 主干，也不要改 auto-label
  主干。

  推荐的数据流：
  routing_labels.jsonl + unified/train.jsonl
  -> 构建 5 个 semantic routes
  -> 对 unified/test query 做 semantic routing
  -> 输出 predicted_label
  -> 复用 simulate_strategy
  -> 复用 metrics

  具体建议：
  - 新增一个适配文件，例如
    llmrouter/src/router/semantic_router_strategy.py
  - 这个文件做两件事：
      - build_semantic_router_from_train()
      - run_semantic_router_inference(query_ids, benchmark_results)
  - route 设计先用最简单的 5 路：
      - qwen2.5-1.5b
      - qwen2.5-3b
      - qwen2.5-7b
      - qwen2.5-14b
      - qwen2.5-32b
  - 每个 route 的 utterances 来自 train split 里对应 routing_label 的 query
  - 推理输出统一成和 classifier 一样的格式：

  predictions[qid] = {
      "id": qid,
      "predicted_label": 2,
  }

  这样可以直接塞进 llmrouter/src/evaluate/run_evaluation.py:425 之后，和 classifier_predictions 并列接入，再复用
  llmrouter/src/evaluate/run_evaluation.py:291。

  最小接入顺序：
  1. 只做 Phase 3 evaluation 集成，不碰在线 runtime。
  2. 只用 unified/train 建 semantic index，严格避免 test 泄漏。
  3. 首版 route 不设阈值，始终返回 top-1 route。
  4. 首版先过滤 all_wrong 训练样本，和 classifier 保持一致。
  5. 先让它产出 predicted_label，不要急着改 metrics。

  为什么这是最优接法
  因为 llmrouter 当前不是“统一 router 框架”，而是“离线 benchmark + 多策略模拟器”。所以最稳的方式不是把 semantic-
  router 硬塞进 benchmark 阶段，而是让它扮演和 classifier 同样的角色：预测 tier。

  换句话说：
  - classifier 的职责：query -> predicted_label
  - semantic-router 的职责也应该是：query -> predicted_label
  只要这一步对齐，后面的评测链条都能复用。

  需要提前注意的风险
  - 路径硬编码很多，当前脚本大量写死 /tangboyan/llmrouter/...，例如 llmrouter/src/evaluate/run_evaluation.py:29、
    llmrouter/src/router/train_classifier.py:28、llmrouter/data/merge_and_split.py:26。如果你在当前 workspace 真
    跑，后面很可能要先做路径归一化。
  - semantic-router 不需要训练，但需要 embedding backend。当前 vendored 目录可导入，但真正跑 dense route 仍需要可
    用 encoder。
  - 当前环境是 Python 3.13。上游 semantic-router[local] 对本地 encoder extras 的支持偏向 <3.13。如果你想走本地
    embedding，最好单独准备兼容环境；如果想快点接评测，优先考虑已有 API/服务型 embedding backend。
  - 现有 Phase 3 不统计 flat router 自身 routing latency，所以 semantic-router 接入后，延迟指标默认仍是“选中模型的
    推理延迟”，不是端到端总延迟。




已完成远程部署和单独评测。
只跑了 semantic-router，没有重训其他 router，也没有重跑整条 pipeline。
接入代码在 llmrouter/src/router/semantic_router_strategy.py:12、llmrouter/src/evaluate/run_evaluation.py:434 和llmrouter/src/analysis/compare_evaluation_summaries.py:1。远程实际使用的是官方本地路线 HuggingFaceEncoder(name="sentence-transformersall-MiniLM-L6-v2")，并在 cuda:0 上验证通过。
原始远程评测入口已备份到 /tangboyan/llmrouter/src/evaluate/run_evaluation.py.bak_20260410_094902。

结果如下。semantic-router 在 unified_test 上：
- Accuracy: 68.18%
- Cost Ratio: 25.9%
- Avg Latency: 857ms
- P99 Latency: 8308ms

和现有方案对比：
- 比 classifier 更准：68.18% vs 66.85%，但成本更高：25.9% vs 21.9%
- 比 cascade-optimized 略低：68.18% vs 68.85%，成本略高：25.9% vs 24.7%
- 但延迟明显优于 cascade-optimized：857ms vs 2053ms，P99 也低很多：8308ms vs 21973ms

远程输出已单独保存到：
- results/evaluation/v2_5tier/semantic_router_gpu/summary.json
- results/evaluation/v2_5tier/semantic_router_gpu/semantic-router_results.jsonl
- results/evaluation/v2_5tier/semantic_router_gpu/comparison.json
- results/evaluation/v2_5tier/semantic_router_gpu/comparison.md




### Router Latency

可能是个**领域空白**：

综述有提到，但只是原则性地提到，不是明确给出“router latency”的单独定义或计算方式。
- 它随后定义了 latency：从发请求到收到响应的时间，并举了 TTFT 和 TPOT。这更像是在讲端到端服务延迟，不是单独讲 router 判路那一步的 latency。

所以结论是：
- 有间接提到 router latency / overhead
- 但没有明确区分：
- router decision latency
- model inference latency
- end-to-end latency
- 也没有说明主实验里是否把 router latency 真正加进最终 latency 指标




当前诊断已经比较明确：
- semantic-router 的 route 严重不平衡。train pool 里，去掉 all_wrong 后各 tier 是 14616 / 4341 / 2391 / 1251 / 717，32b 路由
    太小。
- hardest 样本明显学不好。oracle=4 的 747 条里，只路到 32b 110 条，说明当前方案对高难样本识别不足。
- 主要可疑点有两个：
    1. 把 all_wrong 全部排除了，等于削弱了 hardest route
    2. 大类样本太多，小类样本太少，semantic 空间被低 tier 主导

按这个顺序调，不盲扫参数：
1. 先验证数据分布问题
   先跑 --semantic-include-all-wrong-train
   目的：确认把 hardest 样本补回 route 后，tier=4 识别是否提升
2. 再验证 class imbalance
   再跑 --semantic-include-all-wrong-train --semantic-max-utterances-per-route 2000
   目的：把每个 route 控制到更接近的规模，减少低 tier 对检索空间的支配，同时缩短推理时间
3. 只在前两步有效时再调检索超参
   优先试：
  - --semantic-top-k 10
  - --semantic-aggregation max
    这两项是次级调参，不该在数据分布没处理前先动
4. 评估标准
   不只看总 Accuracy，要同时看：
  - oracle=4 的召回是否提升
  - 总 Cost Ratio 是否失控
  - Avg/P99 Latency 是否下降
  - 是否比 classifier 和 cascade-optimized 更接近 Pareto 前沿
5. 最终决策标准
   我会优先找这样一组配置：
  - 准确率至少超过现在的 68.18%
  - 成本不要明显高过 cascade-optimized 的 24.7%
  - 延迟仍保持明显优于 cascade-optimized



all-MiniLM-L6-v2 这条线我已经跑完 4 组：

  - semantic_router_gpu: 68.18% / 25.9%
  - include_all_wrong: 68.91% / 33.2%
  - include_all_wrong + cap2000: 70.29% / 36.7%
  - include_all_wrong + top5 + max: 62.94% / 20.6%

• 最终结果出来了。
  目前 semantic-router 已完成的关键实验是：
  Accuracy / Cost Ratio：
  - semantic_router_gpu
    68.18% / 25.9%
  - include_all_wrong
    68.91% / 33.2%
  - include_all_wrong + cap2000
    70.29% / 36.7%
  - include_all_wrong + top5 + max
    62.94% / 20.6%
  - bge-m3 + include_all_wrong
    68.72% / 31.8%
  - bge-m3 + include_all_wrong + cap2000
    69.34% / 33.7%


# ***2026.4.11***


## 实践

### 追求极致的正确率：

- 当前 best semantic-router: 79.61% / 97.2%
- 提升是 +0.08 个点 accuracy，同时成本从 100% 降到 97.2%



最新结果：
  跑完了 4 组cost <= 40% 的 semantic-router 新实验。当前最优是：
  - semantic_router_override14b_7b_meta_mpnet_cost40_fresh4xa100
  - Accuracy: 76.28%
  - Cost Ratio: 38.6%
  - Avg Latency: 1054ms
  - P99 Latency: 9744ms

  关键配置：
  - encoder: sentence-transformers/all-mpnet-base-v2
  - routing mode: 32b-override 这一套逻辑被我用作“base model default + semantic override”，这里 base 是 qwen2.5-14b
  - override candidate: qwen2.5-7b
  - text fields: dataset,subject,difficulty
  - tuned threshold: qwen2.5-7b = 0.8628563005596404
  - routing distribution: qwen2.5-14b = 4935, qwen2.5-7b = 1514


### 已知方案总体对比

| 策略 | 准确率 | 相对成本 | 平均延迟 | P99 延迟 |
|------|--------|----------|----------|----------|
| Always 32B | 79.53% | 100.0% | 1539ms | 18354ms |
| Always 14B | 76.73% | 43.7% | 1248ms | 13021ms |
| Always 7B | 72.58% | 22.1% | 659ms | 4878ms |
| Always 3B | 63.92% | 9.4% | 621ms | 4673ms |
| Always 1.5B | 55.93% | 4.7% | 624ms | 4280ms |
| Random | 69.64% | 36.6% | 897ms | 7977ms |
| Oracle | 90.80% | 20.3% | 824ms | 6848ms |
| Cascade (default) | 62.35% | 10.3% | 951ms | 6759ms |
| Cascade (optimized) | 68.85% | 24.7% | 2053ms | 21973ms |
| Binary Gate (logprobs) | 68.85% | 24.7% | 2053ms | 21973ms |
| Classifier (0.5B) | 60.89% | 12.3% | 765ms | 6386ms |
| Semantic Tiered MiniLM cap2000 | 70.71% | 37.2% | 968ms | 9624ms |
| Semantic Tiered MiniLM cost33 | 70.12% | 33.8% | 942ms | 9019ms |
| Semantic Tiered MiniLM cost35.5 | 70.62% | 36.8% | 963ms | 9503ms |
| Semantic Override 32B->14B MPNet | 79.61% | 97.2% | 1525ms | 17398ms |
| Semantic Override 14B->7B MPNet | 76.28% | 38.6% | 1054ms | 9744ms |
| Semantic Override 14B->3B MPNet | 74.57% | 35.7% | 1017ms | 9050ms |
| Semantic Override 14B->1.5B/3B MPNet | 73.67% | 35.1% | 1023ms | 8919ms |


#### 最新 Semantic Router 内部对比

| 方案 | Encoder | 路由形式 | 关键设置 | 准确率 | 成本 |
|------|---------|----------|----------|--------|------|
| Tiered MiniLM cap2000 | all-MiniLM-L6-v2 | 五路 tiered | include_all_wrong + cap2000 | 70.71% | 37.2% |
| Tiered MiniLM tuned cost33 | all-MiniLM-L6-v2 | 五路 tiered | 32B 阈值调优，target cost=0.33 | 70.12% | 33.8% |
| Tiered MiniLM tuned cost35.5 | all-MiniLM-L6-v2 | 五路 tiered | 32B 阈值调优，target cost=0.355 | 70.62% | 36.8% |
| Override 32B->14B MPNet | all-mpnet-base-v2 | 32B 默认，命中后降到 14B | metadata: dataset/subject/difficulty，threshold=0.9364 | 79.61% | 97.2% |
| Override 14B->7B MPNet | all-mpnet-base-v2 | 14B 默认，命中后降到 7B | metadata: dataset/subject/difficulty，threshold=0.8629 | 76.28% | 38.6% |
| Override 14B->3B MPNet | all-mpnet-base-v2 | 14B 默认，命中后降到 3B | metadata: dataset/subject/difficulty，threshold=0.8585 | 74.57% | 35.7% |
| Override 14B->1.5B/3B MPNet | all-mpnet-base-v2 | 14B 默认，命中后降到 1.5B/3B | metadata: dataset/subject/difficulty，threshold=0.8818 | 73.67% | 35.1% |

#### 关键结论
1. **Semantic Router 的当前最高准确率方案**是 `32B 默认 + 14B override + MPNet + metadata`，达到 **79.61% / 97.2%**。相比 Always 32B，准确率仅提高 **0.08** 个百分点，成本下降 **2.8** 个百分点，收益很小，但它证明 semantic override 已经可以做到几乎不掉点。
2. **在 cost <= 40% 约束下，当前最优方案**是 `14B 默认 + 7B override + MPNet + metadata`，达到 **76.28% / 38.6%**。相比 Always 14B，准确率只低 **0.45** 个百分点，但成本少 **5.1** 个百分点。
3. **旧的 tiered semantic-router 已经被 override 方案明显压制。** 最好的 tiered 版本是 `include_all_wrong + cap2000`，只有 **70.71% / 37.2%**；而 `14B->7B override` 在几乎相同成本下把准确率再拉高了 **5.57** 个百分点，成本只增加 **1.4** 个百分点。
4. **真正带来提升的不是“semantic-router”这个名字本身，而是方案形态变化。** 从五路 tiered 改成“强模型默认 + 低一级模型 override”，再叠加 `all-mpnet-base-v2` 和结构化 metadata，效果才明显跃升。
5. **如果目标是把 semantic-router 接到现有 llmrouter pipeline 做单独验证，当前最值得保留的候选只需要两条：**
   - `semantic-override32b-14b-mpnet-meta-acc`：用于验证语义路由的准确率上限。
   - `semantic-override14b-7b-mpnet-meta-cost40`：用于验证成本受限场景下的真实收益。


#### 和已有 Router 方案的相对位置
- 相比 `classifier`，`14B->7B semantic override` 准确率从 **60.89%** 提升到 **76.28%**，但成本也从 **12.3%** 提升到 **38.6%**。
- 相比 `cascade-optimized` / `binary-gate-logprobs` 的 **68.85% / 24.7%**，`14B->7B semantic override` 在准确率上高出 **7.43** 个百分点，但成本也更高。
- 在“可直接上线的简单策略”里，Always 14B 仍然是很强的朴素基线：**76.73% / 43.7%**。Semantic override 的价值在于把这条强基线压到 40% 左右成本时，仍能尽量保留准确率。



## 知识学习

### 参数矩阵 Checkpoint
参数矩阵只保存某些checkpoint：
训练时，**时间换空间**  

为什么 checkpoint 不只存第一层：  
反向传播需要 每一层的输入激活值。如果只存第一层：  
L1 (存)  
L2  
L3  
L4  
反向传播时会反复从 L1 重新算：  
L1→L2→L3  
L1→L2  
...  
计算量会爆炸。因此实际做法是 每隔几层存一个： 
L1 (存)  
L2  
L3  
L4 (存)  
这样反向时最多只需要 重新算中间几层，计算量可控，同时减少显存。




# ***2026.4.12***

## 知识学习

### assignment1-basics/cs336_basics/trainer/utils.py

### assignment1-basics/cs336_basics/model/modules.py

### einsum()

### 常见显卡

### LLM参数量估算

### MoE模型

### PPO/GRPO/DPO

```
# 定义前向传播：给定输入 x，输出线性变换后的结果。 
def forward(self, x: torch.Tensor) -> torch.Tensor:  
    # 用 einsum 实现矩阵乘法。
    # 这里的含义是：
    # 输入 x 的最后一维是 d_in，
    # 权重 weight 的形状是 (d_out, d_in)，
    # 输出的最后一维就变成 d_out。
    return einsum(x, self.weight, '... d_in,  d_out d_in -> ... d_out')
```



# ***2026.4.13***

## 实践

1. Semantic Router 的流程

这版实现入口在 llmrouter/src/router/semantic_router_strategy.py 和 llmrouter/src/evaluate/run_evaluation.py。

```完整流程
1. 从训练集读取已标注样本。
   代码会把 /tangboyan/llmrouter/data/unified/train.jsonl 和 /tangboyan/llmrouter/results/labels/routing_labels.jsonl 对齐，只保留 unified train 里的 query。对应 load_train_labeled_queries()。
2. 把每条训练样本变成 semantic text。
   默认就是 query 本身；如果开了 semantic-text-fields，会把 dataset/subject/difficulty 这类 metadata 也拼进去。对应 build_semantic_text()。
3. 按路由目标组织成 route。
   当前支持两种模式：
    - tiered：5 路分类，直接建 1.5b / 3b / 7b / 14b / 32b 五个 route。
    - 32b-override：不是五路平权，而是“默认强模型 + 若干小模型 override”。例如 14B 默认，7B override。对应 prepare_route_training_records()。
4. 用预训练 encoder 建索引。
   每个 route 里放一批 utterances，semantic-router 用 HuggingFaceEncoder 编码后建立向量索引。对应 build_routes() 和 build_semantic_router_from_train_records()。
5. 推理时对测试 query 编码并检索。
   对测试 query 用同一个 encoder 编码，检索 top-k 相似 utterances，然后按 route 聚合分数。对应 score_routes_for_vector()。
6. 决策。
    - 如果是普通 tiered 且没调阈值：直接取 router 返回的最佳 route。
    - 如果开了 threshold tuning：按阈值判断，没过阈值就 fallback 到默认大模型。
    - 如果是 32b-override：必须走 threshold 逻辑，否则代码直接报错。对应 choose_route_with_thresholds() 和 run_semantic_router_inference()。
7. 评测。
   路由结果不会真实再调模型，而是去查已经离线跑好的 benchmark 结果，看被路由到的模型在该题上是否答对，然后统计 accuracy / cost / latency。对应 simulate_strategy() 和 compute_all_metrics()。
```

  
2. 测试用了什么数据集？ 
测试集是 unified_test，入口写在 llmrouter/src/evaluate/run_evaluation.py。
具体是：
- 测试切分文件：/tangboyan/llmrouter/data/unified/test.jsonl
- 训练切分文件：/tangboyan/llmrouter/data/unified/train.jsonl
- 评测时会读取 5 个模型在各数据集上的 benchmark 结果，再筛出 unified_test 里的 query
- 当前这套 v2_5tier 评测覆盖的数据集，从结果里看是：
    - agieval
    - ceval
    - cmath
    - cmmlu
    - gsm8k
    - logiqa2
当前统一测试集规模是 6449 条。

3. Semantic Router 需要训练吗？ 
结论：不需要像 classifier 那样做参数训练。
- classifier：要训练一个新模型
- semantic-router：不训练新分类器参数，只是“拿预训练 embedding 模型 + 训练集样本建语义路由索引”  

```semantic-router
- 需要一套已标注的训练样本，用来构建 route utterances
- 需要一个预训练 encoder，例如你现在用过的：
    - sentence-transformers/all-MiniLM-L6-v2
    - sentence-transformers/all-mpnet-base-v2
- 可选地需要做一次阈值调优，但这不是模型训练，只是用训练集里再切一小块验证集做搜索  
```

现在这版代码里，threshold tuning 也只是：
- 切一部分 unified_train
- 搜索阈值
- 选 accuracy/cost 最优点
不是 gradient finetune。




## 知识学习

### 为什么不直接存“参数矩阵的转置”？

Y = X x W^T  

你可能会问：既然都要转置，为什么不直接把 self.weight 定义成 (in_features, out_features)？

答案是：为了计算效率（和历史习惯）。
逻辑直观：在 (out, in) 的存储方式下，weight[0]（矩阵的第一行）直接对应于第一个输出神经元的所有权重。这在逻辑上非常清晰。
算子优化：底层硬件（如 NVIDIA GPU）在执行 Linear 算子时，针对这种存储方式做了深度优化。


### 常见数据类型详解

通过浮点数的三个组成部分来理解它们： 
符号位（Sign）、指数位（Exponent，决定范围）和尾数位（Fraction/Mantissa，决定精度）。

#### FP32 (Full Precision / Single Precision)  
结构： 1位符号，8位指数，23位尾数。  
特点： 精度极高，数值范围广。 
LLM 中的角色： 曾经是标准。但在如今的 LLM 训练中，它通常只作为“主权重（Master Weights）”存在，用来在优化器更新时保持微小的梯度变化。

#### FP16 (Half Precision)  
结构： 1位符号，5位指数，10位尾数。  
优点： 内存占用减半，计算速度极快。  
缺点： 数值范围窄（最大约 65504）。在训练 LLM 时，极易产生“梯度溢出（Overflow）”或“下溢（Underflow）”，导致训练崩溃。  
对策： 需要使用混合精度训练（Mixed Precision Training）和损失缩放（Loss Scaling）。

#### BF16 (Brain Floating Point 16) —— LLM 的宠儿  
结构： **1**位符号，**8**位指数，**7**位尾数。  
特点： 它是 Google 为了深度学习专门设计的。它的指数位与 FP32 一样长。  
为什么好用： 它的精度（尾数）虽然不如 FP16，但它的数值范围（Range）和 FP32 完全一样。  
意义： 在训练 LLM 时，你不需要担心梯度溢出，不需要搞复杂的 Loss Scaling。目前主流的大模型（Llama 3, GPT-4 等）基本都采用 BF16 进行预训练。  


### einsum()
通过 einsum，即使输入是一个高维张量（例如 x 的形状是 (batch_size, L, d_model)），我们仍然可以通过 广播 规则来进行矩阵乘法（在这种情况下，广播会自动应用到批次维度和其他维度）。 
所以，即使 x 不是二维矩阵，einsum 也能处理高维张量并正确地进行矩阵运算，保证维度匹配。  





# ***2026.4.14***

## 知识学习

### vLLM Semantic Router 
是一个面向 多模型系统 的“语义路由与运行控制层”，不是单纯的模型网关，也不是只做学术路由实验的分类器。 
它的官方定位是：在云、数据中心、边缘侧，为 Mixture-of-Models 提供系统级智能路由。README.md
- 不同模型在能力、成本、延迟、隐私边界上差异很大，单一模型很难覆盖所有流量。
- 真实请求不仅要“选模型”，还要同时处理安全、缓存、记忆、RAG、工具调用、回放审计等系统能力。
- 路由逻辑不能只停留在一个分类器上，而要变成可配置、可验证、可部署、可观测的运行时系统。 
这个项目本质上更像一个 LLM 流量控制平面。它位于客户端和后端模型之间，理解请求，再决定走哪条路、用哪个模型、是否启用插件能力、是否需要额外的安全或工具策略。README.md docs/agent/repo-map.md


#### 系统架构
把“路由”拆成了几个清晰层次，而不是用一个黑盒分类器直接输出模型名：
- signal evaluation
- projection coordination
- decision selection
- model selection
- plugin handling
在 AMD 参考 profile 里，这条链路写得很明确：先做多种信号检测，再做投影/分区，再选路由决策，最后把请求转发到对应模型别名。deploy/amd/README.md

- Signals：检测层。定义“识别到了什么”。支持关键词、语言、上下文长度、结构、权限、embedding、domain、complexity、fact-check、jailbreak、PII、preference、reask、user-feedback、knowledge base 等。website/docs/tutorials/signal/overview.md
- Projections：协调层。把多个弱信号合成为可复用的中间事实，比如 intent partition、difficulty band、verification_required 这类 band，而不是把数值逻辑散落在每个 route 里。website/docs/tutorials/projection/overview.md
- Decisions：策略层。用布尔规则、优先级、tier 选出一条 route。这里是“哪条策略赢”。website/docs/tutorials/decision/overview.md src/semantic-router/pkg/config/decision_config.go
- Algorithms / Model Selection：候选模型选择层。一个 decision 可以挂多个候选模型，再用静态或学习式算法选最优，包括 static、elo、router_dc、automix、hybrid、rl_driven、gmtrouter、latency_aware，以及 looper 类的 confidence、ratings、remom。config/README.md src/semantic-router/pkg/extproc/req_filter_classification_runtime.go src/semantic-router/pkg/modelselection/selector.go
- Plugins：路由后处理层。匹配到某条 route 后，可以附加 route-local 行为，比如 semantic cache、RAG、memory、router replay、tools、system prompt、request params、content safety、hallucination、response jailbreak、image generation 等。website/docs/tutorials/plugin/overview.md

不只是“把问题分类到模型”，而是在做 信号驱动的策略编排:  
比如可以先识别“这是法律高风险请求”，再叠加“需要核验来源”“上下文很长”“用户在追问纠错”，最后才决定走 premium specialist 路线，并启用相应插件。


#### 配置与运行方式
这个项目的另一大特点是配置体系比较完整，而且是统一的。
它采用一套 canonical YAML 合同：
- version
- listeners
- providers
- routing
- global
其中：
- routing 负责语义路由本身，包括 modelCards、signals、projections、decisions
- providers 负责具体部署绑定和默认模型
- global 负责全局运行时能力，比如 observability、router replay、stores、tools、looper、modelcatalog 等。这套约定写在公开配置文档里，也被仓库测试强约束。website/docs/installation/configuration.md configREADME.md  

此外，这个项目同时支持两种配置视角：
- YAML canonical config
- DSL authoring surface
也就是说，用户既可以直接写 config.yaml，也可以用 DSL/可视化编辑器去表达路由图，然后再编译回canonical YAML。这让它既适合工程部署，也适合调参和策略设计。
website/docs/installation/configuration.md

在部署侧，它不是单一路径，而是支持多种环境：
- 本地 CPU 开发
- 本地 AMD/ROCm 开发
- Kubernetes / Helm / Operator
- Dashboard 控制台
- E2E profile 驱动的测试环境

仓库文档给出的本地默认流程是：
- make vllm-sr-dev
- vllm-sr serve --image-pull-policy never
对应 CPU / AMD 两套本地环境说明也很清楚。docs/agent/environments.md


#### 仓库组成
从代码组织上看，这个仓库已经不是一个单体 router，而是一整套平台： 
- src/semantic-router：Go 核心路由器，包含 config、classification、decision engine、Envoy extproc、selection、plugin runtime。
- src/vllm-sr：Python CLI，负责本地启动、配置校验、Docker 编排、开发体验。
- dashboard：前后端控制台，用于配置编辑、部署、状态查看、playground、可视化。
- deploy/operator：Kubernetes Operator 和 CRD。
- deploy/helm：Helm chart。
- src/training：模型选择与分类相关训练脚本、数据、推理服务。
- e2e：端到端测试框架，覆盖 routing、safety、cache、response-api、dashboard、authz、streaming 多 profile。
- candle-binding ml-binding nlp-binding：Rust/native bindings，用于更底层的推理或 ML 能力接入。


```架构图
  Authoring / Control Plane
    Dashboard / DSL / YAML / CLI / Helm / Operator
          |
          v
  Canonical Config v0.3
    version / listeners / providers / routing / global
          |
          v
  Runtime Plane
    Client
      -> Envoy
      -> semantic-router extproc (OpenAIRouter)
         -> Signals
         -> Projections
         -> Decisions
         -> Algorithms / Looper
         -> Route-local Plugins
         -> Provider binding / endpoint selection / alias rewrite
         -> Upstream model backends
      <- Response filters / replay / cache / warnings / headers
          |
          v
  Observability / Replay / Dashboard Insight

  Validation / Support Plane
    E2E profiles / deploy recipes / training stack / Rust-native bindings
```
这张图背后的关键点是：
- 这套系统有一个统一配置合同，不是 CLI 一套、Dashboard 一套、Operator 一套。仓库明确把入口统一为 version / listeners / providers / routing / global，其中 routing 负责 `modelCards5), website/docs/tutorials/projection/overview.md:9, website/docs/tutorials/decision/overview.md:7, website/docs/tutorials/algorithm/overview.md:7, website/docs/tutorials/plugin/overview.md:5, deploy/amd/README.md:100)
- 仓库形态也说明它是平台，不是单一 router binary。src/semantic-router 是 Go 路由内核，src/vllm-sr 是 Python CLI，dashboard/ 是控制台，deploy/operator/ 和 deploy/helm/ 是 K8s 部署面，e2e/ 是验证框架，src/training/ 和 Rust bindings 是算法/模型支持层。(docs/agent/repo-map.md:3)

所以一句话说，它更像“LLM 流量控制平面 + 运行时策略编排层”，而不是“模型网关 + 少量规则”。



#### 一次请求怎么被路由
1. 启动阶段先由 vllm-sr serve 做 bootstrap，解析配置、选择 Docker/K8s backend、准备 runtime config，然后把本地或集群拓扑拉起来。(src/vllm-sr/cli/commands/runtime.py:57, src/vllm-sr/cli/commands/runtime.py:214)  
2. 真正请求进入时，Go 侧的 OpenAIRouter 作为 Envoy extproc server 工作。它不是只处理 request body，而是完整跑四个阶段：request headers -> request body -> response headers -> response body。(src/semantic-router/pkg/extproc/router.go:24, src/semantic-router/pkg/extproc/processor_core.go:48)
3. request headers 阶段会先抓 request_id、:path、:method、streaming 预期、looper 内部请求标记等。也就是说，这里先决定“这是普通 chat、Response API、models 接口，还是 looper 内部调用”。(src/semantic-router/pkg/extproc/processor_req_header.go:17)
4. request body 阶段先走一个快路径：如果是 Response API，就先翻译成 chat completions 形态；然后做 body 校验；再用 fast extractor 直接拿到 model / userContent / firstImageURL / stream，避免一开始就完整反序列化。(src/semantic-router/pkg/extproc/processor_req_body.go:22, /home/ryan/CUHKSZ/LLM-Router/V:61, src/semantic-router/pkg/decision/engine.go:60, src/semantic-router/pkg/decision/engine.go:199)
5. decision engine 本身是个布尔规则树求值器。叶子节点是 type + name，支持 AND / OR / NOT，命中后会得到 confidence；多个 decision 都命中时，再按 tier -> confidence -> priority 或 priority -> confidence 选出最终 route。(src/semantic-router/pkg/config/decision_config.go:3, src/semantic-router/pkg/decision/engine.go:151, src/semantic-router/pkg/decision/engine.go:335)
6. route 选出来以后，不一定立刻等于“最终模型已定”。
如果用户显式指定模型，router 会保留原模型，但仍然保留 decision 结果给插件使用。
如果用户走的是 auto model，router 才会根据 decision.modelRefs + decision.algorithm 去做候选选择。(src/semantic-router/pkg/extproc/req_filter_classification_runtime.go:138, src/semantic-router/pkg/extproc/req_filter_classification.go:61)
7. 候选模型选择分两类。单模型选择算法走 selector registry，比如 static / elo / router_dc / automix / hybrid / rl_driven / gmtrouter / latency_aware / knn / kmeans / svm。多模型编排算法走 looper，比如 confidence / ratings / remom。(website/docs/tutorials/algorithm/overview.md:55, src/semantic-router/pkg/selection/factory.go:96, src/semantic-router/pkg/extproc/req_filter_looper.go:45) 
8. 在真正发往上游前，router 还会跑一组 route-local 行为：fast_response、rate limit、semantic cache short-circuit、RAG 检索、modality 处理、memory 注入、request params、system prompt、tools 选择。然后才做 endpoint 选择、alias 到 provider-specific model id 的映射，并把修改后的 body 发给上游。(src/semantic-router/pkg/extproc/processor_req_body_prepare.go:63, src/semantic-router/pkg/extproc/req_filter_rag.go:19, src/semantic-router/pkg/extproc/processor_req_body_routing.go:28, src/semantic-router/pkg/extproc/processor_req_body_routing.go:65, /home/ryan/CUHKSZ/LLM-Router/VLLM-sem)

把这 12 步压成一句话就是：
客户端只发出一次 OpenAI 兼容请求，但 router 在内部实际完成了:  
“请求理解、信号抽取、投影协调、策略命中、候选模型选择、插件执行、后端绑定、响应审计与告警”  
这整条系统链路。  




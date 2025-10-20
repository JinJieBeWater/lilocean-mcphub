# 小海物联网智能终端 - 软件部署文档

## 项目概述

小海是基于 xiaozhi 生态的物联网智能终端，运行在 ESP32 上，采用流式 ASR + LLM + TTS 架构的语音交互系统。本部署文档涵盖从 Git 拉取仓库到本地 Docker 部署的完整流程。

## 系统架构

- **前端**: React + Vite + Tailwind CSS
- **后端**: Node.js + Express
- **数据库**: PostgreSQL + pgvector
- **容器化**: Docker + Docker Compose

## 环境要求

### 硬件要求
- 内存: 4GB 以上
- 存储: 10GB 可用空间
- 网络: 稳定的互联网连接

### 软件要求
- Docker Engine 20.10+
- Docker Compose 2.0+
- Git

## 部署步骤

### 1. 克隆项目仓库

```bash
# 克隆项目到本地
git clone https://github.com/JinJieBeWater/lilocean-mcphub.git
cd lilocean-mcphub

# 检查项目结构
ls -la
```

### 2. Docker 部署

#### 使用 Docker Compose 一键部署

```bash
# 启动所有服务
docker compose up -d

# 查看服务状态
docker compose ps
```

#### 服务说明

启动的服务包括：

- **db**: PostgreSQL + pgvector 数据库服务
  - 端口: 5432 (仅本地访问)
  - 数据卷: pgdata

- **mcphub**: 主应用服务
  - 端口: 3000
  - 数据卷: appdata
  - 依赖: db 服务健康状态

- **homeassistant**: Home Assistant 服务（可选）
  - 网络模式: host
  - 用于集成米家设备控制

#### 验证部署

```bash
# 检查容器状态
docker ps

# 检查应用健康状态
curl http://localhost:3000/health

# 检查数据库连接
docker exec -it lilocean-pgvector psql -U lilocean -d lilocean_mcphub -c "SELECT version();"
```

### 3. 获取小海的 websocket 连接连接端点

#### 访问 Xiaozhi 控制面板

1. 打开浏览器访问: `https://xiaozhi.me/` | 或者 自己的小智后台地址
2. 打开控制台
3. 复制 WebSocket 连接端点，后续所有 mcp 服务都需要通过这个端点接入

### 4. 通用 MCP 服务接入

#### 访问 Xiaozhi-MCPHub 控制面板

1. 打开浏览器访问: `http://localhost:3000`
2. 使用默认管理员账户登录:
   - 用户名: `admin`
   - 密码: `admin123`

#### 修改管理员密码

首次登录后，请立即修改管理员密码：
1. 进入 Settings 页面
2. 点击 Change Password
3. 设置新的安全密码

#### 配置 MCP 服务器

在控制台中添加小智设备：

1. 点击左侧小智导航
2. 添加端点，输入 `WebSocket URL`，后点击`创建`

在控制台添加 MCP 服务器：

1. 访问魔塔 `https://modelscope.cn/mcp` 或其他社区 MCP 服务
2. 选择想要的MCP服务进入
3. 查看右侧的服务配置信息 Streamable HTTP/SSE/Stdio 三种方式都支持
4. 按需求修改配置后点击`连接`
5. 复制配置信息
6. 访问 `http://localhost:3000/servers` 点击添加，按照连接方式与配置填入信息后点击添加
7. 点击刷新按钮，待添加服务状态改为`在线`后，即添加成功

### 5. 借助 Home Assistant 接入米家 MCP 服务

#### 配置 Home Assistant

分为三步：
1. 创建 Home Assistant 账号
2. 安装并配置 HACS
3. 米家设备接入
4. 安装 ha-mcp-for-xiaozhi

#### 创建 Home Assistant 账号

1. 打开 Home Assistant 控制面板 `http://localhost:8123`
1. 点击 `创建我的智能家居`，按照引导创建账号，选择家的位置后进入控制台

#### 安装并配置 HACS

HACS（Home Assistant Community Store）为 Home Assistant 的应用商店
通过 HACS 我们能够安装第三方集成来增强 Home Assistant 的功能。后续我们需要通过 HACS 来安装小米官方的 Home Assistant 集成。

1. 终端输入 `docker exec -it lilocean-homeassistant bash` 进入容器
2. 输入 `wget -O - https://get.hacs.xyz | bash -` 安装 HACS
3. 安装完成后，在项目根目录下打开终端 `docker compose restart` 重启服务
4. 重写进入 Home Assistant 控制面板 `http://localhost:8123`
5. 在`设置`中打开`设备与服务`，点击`添加集成`，选择`HACS` 并添加，按流程完成集成
6. 此时左侧导航出现`HACS`，点击即可进入 HACS 页面

#### 米家设备接入

1. 打开 Home Assistant 控制面板 `http://localhost:8123`
2. 进入 HACS 页面，搜索 `Xiaomi Home`。
3. 点击下载
4. 下载完成后，在项目根目录下打开终端 `docker compose restart` 重启服务
5. 在`设置`中打开`设备与服务`，点击`添加集成`，选择`Xiaomi Home`并添加，按流程完成集成
6. 注意：遇到页面跳转到`homeassistant.local:8123`时，将 `homeassistant.local` 替换为`localhost`即可

#### 安装 ha-mcp-for-xiaozhi

1. 打开 Home Assistant 控制面板 `http://localhost:8123`
2. 进入 HACS 页面，搜索 `ha-mcp-for-xiaozhi`。
3. 点击下载
4. 下载完成后，在项目根目录下打开终端 `docker compose restart` 重启服务
5. 在`设置`中打开`设备与服务`，点击`添加集成`，选择`ha-mcp-for-xiaozhi`并添加
6. 输入设备的 `websocket` 连接地址并确认
7. 在`设置`中打开`语音助手`，调整语言与暴露给设备的实体

### 完成软件部署

现在小海已经可以访问到云端的 MCP 服务


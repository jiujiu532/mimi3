# mimi3 (mimo2api)

小米 AI Studio 自动化控制网关，将 MIMO 模型进行转发并兼容 OpenAI API 格式。

## 功能

- OpenAI 兼容 API 中转（支持 `/v1/chat/completions`, `/v1/responses`, `/anthropic/v1/messages`）
- Web 控制面板（实时监控、日志查看）
- 多账号轮询负载均衡
- 流式响应支持
- 自动管理 Claw 实例生命周期（55 分钟自动销毁重建）

## Docker 部署（推荐）

### 1. 拉取镜像

```bash
docker pull ghcr.io/jiujiu532/mimi3:latest
```

### 2. 准备配置

```bash
# 创建目录
mkdir -p mimi3/users && cd mimi3

# 创建环境变量文件
cat > .env << 'EOF'
# 网关绑定地址与端口
SERVER_HOST=0.0.0.0
SERVER_PORT=8000

# [重要] Claw 节点连接的 WebSocket 桥接地址
# 必须设置为你的公网域名或IP，Claw 内网的 bridge 脚本通过此地址连回网关
WS_TUNNEL_URL=ws://your-domain.com:8000/ws

# 本机 OpenAI 兼容 /v1 的 Bearer 密钥（客户端 api_key）；不设则不对中转鉴权
# MIMO_RELAY_OPENAI_KEY=sk-your-random-secret-here

# WebUI 登录口令；不设则不启用 WebUI/API 管理面登录
# MIMO_WEBUI_USERNAME=admin
# MIMO_WEBUI_PASSWORD=change-me
EOF
```

### 3. 添加账号

在 `users/` 目录下创建 JSON 文件（文件名格式 `user_xxx.json`）：

```json
{
  "userId": "你的userId",
  "serviceToken": "你的serviceToken",
  "xiaomichatbot_ph": "你的xiaomichatbot_ph",
  "name": "账号备注名"
}
```

获取方式：前往 https://aistudio.xiaomimimo.com 登录后，从浏览器开发者工具复制 Cookie 中的 `serviceToken`、`userId`、`xiaomichatbot_ph`。

### 4. 启动

```bash
# 使用 docker-compose
docker compose up -d

# 或直接 docker run
docker run -d \
  --name mimi3 \
  --restart unless-stopped \
  -p 8000:8000 \
  -v ./users:/app/users \
  -v ./logs:/app/logs \
  -v ./.env:/app/.env:ro \
  -e TZ=Asia/Shanghai \
  ghcr.io/jiujiu532/mimi3:latest
```

### 5. 使用

API 地址：`http://your-server:8000/v1/chat/completions`

WebUI 面板：`http://your-server:8000/`

## 手动部署

```bash
# 安装依赖
pip install -r requirements.txt

# 复制并配置环境变量
cp env.example .env

# 启动服务
python main.py
```

## 技术栈

Python / FastAPI / Uvicorn / WebSocket

## 免责声明

1. **本项目仅供学习交流使用，禁止一切商业/滥用行为。**
2. 本项目为个人独立开发的开源项目，与小米公司及其关联方**无任何隶属、授权或合作关系**。
3. MIMO、Xiaomi AI Studio 等名称及商标归小米公司所有，本项目不主张任何权利。
4. 本项目不提供任何小米账号、密钥或付费服务的破解，仅作为技术研究用途。
5. 使用者应遵守所在地法律法规及小米服务条款，因使用本项目产生的一切后果由使用者自行承担。
6. 本项目代码随缘更新，作者不提供任何保证或技术支持。
7. **建议优先使用小米官方 API**，本项目仅为技术研究备选方案。
8. 如有任何权益问题，请联系删除。

## 致谢
[linux.do](https://linux.do)

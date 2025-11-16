# TrendRadar Docker 部署配置说明

## 📅 发送频率配置

### 方式一：使用环境变量（推荐）

在 `docker-compose.yml` 同级目录创建 `.env` 文件：

```bash
# 每5分钟执行一次
CRON_SCHEDULE=*/5 * * * *

# 或者每30分钟执行一次
# CRON_SCHEDULE=*/30 * * * *

# 或者每小时整点执行
# CRON_SCHEDULE=0 * * * *
```

### 方式二：直接在 docker-compose.yml 中配置

`docker-compose.yml` 中已经默认设置为每5分钟执行一次：

```yaml
environment:
  - CRON_SCHEDULE=${CRON_SCHEDULE:-*/5 * * * *}
```

如果不创建 `.env` 文件，将使用默认值 `*/5 * * * *`（每5分钟）。

## 🚀 启动服务

```bash
# 进入 docker 目录
cd docker

# 启动服务（如果使用 .env 文件，确保已创建）
docker-compose up -d

# 查看日志
docker logs -f trend-radar

# 查看运行状态
docker exec -it trend-radar python manage.py status
```

## ⚙️ Cron 表达式说明

格式：`分钟 小时 日 月 星期`

常用示例：
- `*/5 * * * *` - 每5分钟执行一次
- `*/30 * * * *` - 每30分钟执行一次
- `0 * * * *` - 每小时整点执行
- `0 9,12,18 * * *` - 每天9点、12点、18点执行
- `0 9-18 * * 1-5` - 工作日9点到18点，每小时整点执行

## 📝 完整 .env 文件示例

```bash
# ==================== 发送频率配置 ====================
CRON_SCHEDULE=*/5 * * * *
RUN_MODE=cron
IMMEDIATE_RUN=true

# ==================== 通知渠道配置 ====================
FEISHU_WEBHOOK_URL=你的飞书webhook地址
TELEGRAM_BOT_TOKEN=你的Telegram Bot Token
TELEGRAM_CHAT_ID=你的Telegram Chat ID
DINGTALK_WEBHOOK_URL=你的钉钉webhook地址
WEWORK_WEBHOOK_URL=你的企业微信webhook地址

# 邮件配置
EMAIL_FROM=发件人邮箱
EMAIL_PASSWORD=邮箱密码或授权码
EMAIL_TO=收件人邮箱

# ntfy配置
NTFY_SERVER_URL=https://ntfy.sh
NTFY_TOPIC=你的ntfy主题名称
```

## 🔍 验证配置

启动容器后，可以通过以下命令验证配置：

```bash
# 查看容器状态和cron配置
docker exec -it trend-radar python manage.py status

# 查看详细配置
docker exec -it trend-radar python manage.py config

# 手动执行一次测试
docker exec -it trend-radar python manage.py run
```

## ⚠️ 注意事项

1. **修改配置后需要重启容器**：
   ```bash
   docker-compose restart
   ```

2. **时区设置**：默认使用 `Asia/Shanghai`，可在 `docker-compose.yml` 中修改 `TZ` 环境变量

3. **不使用 GitHub Actions**：如果使用 Docker 部署，GitHub Actions 不会自动运行（除非你手动触发）


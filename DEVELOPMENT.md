# TrendRadar 开发调试指南

## 📋 前置准备

### 1. 安装 Python 环境

- Python 3.10 或更高版本
- 推荐使用虚拟环境

### 2. 安装依赖

#### 方式一：使用 UV（推荐，项目使用 UV 管理）

**Windows:**
```bash
# 运行自动安装脚本
setup-windows.bat

# 或手动安装 UV
pip install uv
uv sync
```

**Linux/macOS:**
```bash
# 安装 UV
curl -LsSf https://astral.sh/uv/install.sh | sh

# 安装项目依赖
uv sync
```

#### 方式二：使用 pip

```bash
pip install -r requirements.txt
```

### 3. 准备配置文件

确保以下文件存在：

- `config/config.yaml` - 主配置文件
- `config/frequency_words.txt` - 关键词配置文件

如果不存在，可以从项目模板复制或参考 `readme.md` 创建。

## 🚀 开发调试启动方式

### 方式一：直接运行主程序（推荐用于开发调试）

这是最简单的开发调试方式，程序会：
- 执行一次完整的爬取和分析流程
- 在本地环境自动打开浏览器显示结果
- 输出详细的日志信息

**Windows:**
```bash
# 使用 UV 运行
uv run python main.py

# 或直接使用 Python（如果已安装依赖）
python main.py
```

**Linux/macOS:**
```bash
# 使用 UV 运行
uv run python main.py

# 或直接使用 Python
python3 main.py
```

**输出说明：**
- 程序会在 `output/` 目录生成 HTML 和 TXT 报告
- 如果配置了推送渠道，会发送通知
- 在本地环境会自动打开浏览器显示报告

### 方式二：使用环境变量配置（调试不同配置）

**Windows PowerShell:**
```powershell
# 设置环境变量
$env:FEISHU_WEBHOOK_URL="你的webhook地址"
$env:REPORT_MODE="incremental"  # 测试增量模式
$env:ENABLE_NOTIFICATION="true"

# 运行程序
uv run python main.py
```

**Windows CMD:**
```cmd
set FEISHU_WEBHOOK_URL=你的webhook地址
set REPORT_MODE=incremental
set ENABLE_NOTIFICATION=true

python main.py
```

**Linux/macOS:**
```bash
export FEISHU_WEBHOOK_URL="你的webhook地址"
export REPORT_MODE="incremental"
export ENABLE_NOTIFICATION="true"

uv run python main.py
```

### 方式三：使用 Docker 单次执行模式（测试 Docker 环境）

```bash
cd docker

# 单次执行模式（不启动定时任务）
docker run --rm -it \
  -v ./config:/app/config:ro \
  -v ./output:/app/output \
  -e RUN_MODE=once \
  -e FEISHU_WEBHOOK_URL="你的webhook地址" \
  wantcat/trendradar:latest
```

### 方式四：启动 MCP 服务器（用于 AI 分析功能）

**HTTP 模式（适合调试和远程访问）:**

**Windows:**
```bash
start-http.bat
# 或手动运行
uv run python -m mcp_server.server --transport http --port 3333
```

**Linux/macOS:**
```bash
./start-http.sh
# 或手动运行
uv run python -m mcp_server.server --transport http --port 3333
```

**STDIO 模式（用于 MCP 客户端连接）:**
```bash
uv run python -m mcp_server.server --transport stdio
```

## 🔍 调试技巧

### 1. 查看详细日志

程序会输出详细的执行日志，包括：
- 配置加载信息
- 爬取进度
- 数据处理结果
- 推送状态

### 2. 检查配置文件

程序启动时会检查配置文件：
```python
# 如果配置文件缺失，会显示错误提示
❌ 配置文件错误: FileNotFoundError
请确保以下文件存在:
  • config/config.yaml
  • config/frequency_words.txt
```

### 3. 测试不同模式

通过环境变量或配置文件切换模式：

```bash
# 增量模式（只推送新增）
export REPORT_MODE=incremental
python main.py

# 当前榜单模式
export REPORT_MODE=current
python main.py

# 当日汇总模式（默认）
export REPORT_MODE=daily
python main.py
```

### 4. 禁用推送（仅测试爬取功能）

在 `config/config.yaml` 中设置：
```yaml
notification:
  enable_notification: false
```

或使用环境变量：
```bash
export ENABLE_NOTIFICATION=false
python main.py
```

### 5. 查看生成的报告

程序会在 `output/` 目录生成报告：
```
output/
├── 2025-01-15/
│   ├── html/
│   │   └── report_20250115_123045.html
│   └── txt/
│       └── report_20250115_123045.txt
```

### 6. 测试推送渠道

**测试飞书推送：**
```bash
export FEISHU_WEBHOOK_URL="你的webhook地址"
python main.py
```

**测试邮件推送：**
```bash
export EMAIL_FROM="发件人邮箱"
export EMAIL_PASSWORD="密码或授权码"
export EMAIL_TO="收件人邮箱"
python main.py
```

## 🐛 常见问题排查

### 问题1：配置文件不存在

**错误信息：**
```
❌ 配置文件错误: FileNotFoundError
```

**解决方法：**
1. 检查 `config/config.yaml` 是否存在
2. 检查 `config/frequency_words.txt` 是否存在
3. 确保在项目根目录运行程序

### 问题2：依赖缺失

**错误信息：**
```
ModuleNotFoundError: No module named 'xxx'
```

**解决方法：**
```bash
# 重新安装依赖
uv sync
# 或
pip install -r requirements.txt
```

### 问题3：浏览器未自动打开

**原因：**
- 在 Docker 或 GitHub Actions 环境中运行（这是正常的）
- 浏览器配置问题

**解决方法：**
- 手动打开 `output/日期/html/report_*.html` 文件
- 检查 `_should_open_browser()` 方法的逻辑

### 问题4：推送失败

**检查步骤：**
1. 验证 webhook URL 是否正确
2. 检查网络连接
3. 查看程序输出的错误信息
4. 检查配置文件中的推送设置

## 📝 开发建议

1. **使用虚拟环境**：避免依赖冲突
2. **配置 Git 忽略**：不要提交包含敏感信息的配置文件
3. **分步测试**：先测试爬取，再测试推送
4. **查看日志**：程序输出详细的日志信息，有助于调试
5. **使用 IDE 调试**：可以在 IDE 中设置断点进行调试

## 🔗 相关文档

- [README.md](readme.md) - 项目主文档
- [docker/README.md](docker/README.md) - Docker 部署文档
- [README-MCP-FAQ.md](README-MCP-FAQ.md) - MCP 功能使用文档


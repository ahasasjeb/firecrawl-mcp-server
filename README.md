# Firecrawl MCP 服务器

一个与 [Firecrawl](https://github.com/mendableai/firecrawl) 集成的模型上下文协议 (MCP) 服务器实现，提供网页抓取功能。

特别感谢 [@vrknetha](https://github.com/vrknetha) 和 [@cawstudios](https://caw.tech) 的初始实现！

## 功能

- 支持抓取、爬取、搜索、提取、深度研究和批量抓取
- 支持 JavaScript 渲染的网页抓取
- URL 发现和爬取
- 带内容提取的网页搜索
- 自动重试，采用指数退避策略
- 高效的批量处理，内置速率限制
- 云 API 的信用使用监控
- 全面的日志系统
- 支持云和自托管的 FireCrawl 实例
- 移动/桌面视口支持
- 智能内容过滤，支持标签包含/排除

## 安装

### 使用 npx 运行

```bash
env FIRECRAWL_API_KEY=fc-YOUR_API_KEY npx -y firecrawl-mcp
```

### 手动安装

```bash
npm install -g firecrawl-mcp
```

### 在 Cursor 上运行

配置 Cursor 🖥️
注意：需要 Cursor 版本 0.45.6+

要在 Cursor 中配置 FireCrawl MCP：

1. 打开 Cursor 设置
2. 转到 Features > MCP Servers
3. 点击 "+ Add New MCP Server"
4. 输入以下内容：
   - 名称："firecrawl-mcp"（或您喜欢的名称）
   - 类型："command"
   - 命令：`env FIRECRAWL_API_KEY=your-api-key npx -y firecrawl-mcp`

> 如果您使用的是 Windows 并遇到问题，请尝试 `cmd /c "set FIRECRAWL_API_KEY=your-api-key && npx -y firecrawl-mcp"`

将 `your-api-key` 替换为您的 FireCrawl API 密钥。

添加后，刷新 MCP 服务器列表以查看新工具。Composer Agent 将在适当时自动使用 FireCrawl MCP，但您可以通过描述您的网页抓取需求来显式请求它。通过 Command+L（Mac）访问 Composer，选择提交按钮旁边的 "Agent"，然后输入您的查询。

### 在 Windsurf 上运行

将此添加到您的 `./codeium/windsurf/model_config.json`：

```json
{
  "mcpServers": {
    "mcp-server-firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY_HERE"
      }
    }
  }
}
```

### 通过 Smithery 安装（旧版）

要通过 [Smithery](https://smithery.ai/server/@mendableai/mcp-server-firecrawl) 自动安装 FireCrawl 以用于 Claude Desktop：

```bash
npx -y @smithery/cli install @mendableai/mcp-server-firecrawl --client claude
```

## 配置

### 环境变量

#### 云 API 必需

- `FIRECRAWL_API_KEY`：您的 FireCrawl API 密钥
  - 使用云 API 时必需（默认）
  - 使用带有 `FIRECRAWL_API_URL` 的自托管实例时可选
- `FIRECRAWL_API_URL`（可选）：自托管实例的自定义 API 端点
  - 示例：`https://firecrawl.your-domain.com`
  - 如果未提供，将使用云 API（需要 API 密钥）

#### 可选配置

##### 重试配置

- `FIRECRAWL_RETRY_MAX_ATTEMPTS`：最大重试次数（默认：3）
- `FIRECRAWL_RETRY_INITIAL_DELAY`：首次重试前的初始延迟（毫秒）（默认：1000）
- `FIRECRAWL_RETRY_MAX_DELAY`：重试之间的最大延迟（毫秒）（默认：10000）
- `FIRECRAWL_RETRY_BACKOFF_FACTOR`：指数退避乘数（默认：2）

##### 信用使用监控

- `FIRECRAWL_CREDIT_WARNING_THRESHOLD`：信用使用警告阈值（默认：1000）
- `FIRECRAWL_CREDIT_CRITICAL_THRESHOLD`：信用使用临界阈值（默认：100）

### 配置示例

使用云 API 并自定义重试和信用监控：

```bash
# 云 API 必需
export FIRECRAWL_API_KEY=your-api-key

# 可选重试配置
export FIRECRAWL_RETRY_MAX_ATTEMPTS=5        # 增加最大重试次数
export FIRECRAWL_RETRY_INITIAL_DELAY=2000    # 从 2 秒延迟开始
export FIRECRAWL_RETRY_MAX_DELAY=30000       # 最大 30 秒延迟
export FIRECRAWL_RETRY_BACKOFF_FACTOR=3      # 更积极的退避

# 可选信用监控
export FIRECRAWL_CREDIT_WARNING_THRESHOLD=2000    # 在 2000 信用时警告
export FIRECRAWL_CREDIT_CRITICAL_THRESHOLD=500    # 在 500 信用时临界
```

使用自托管实例：

```bash
# 自托管必需
export FIRECRAWL_API_URL=https://firecrawl.your-domain.com

# 自托管的可选身份验证
export FIRECRAWL_API_KEY=your-api-key  # 如果您的实例需要身份验证

# 自定义重试配置
export FIRECRAWL_RETRY_MAX_ATTEMPTS=10
export FIRECRAWL_RETRY_INITIAL_DELAY=500     # 从更快的重试开始
```

### 在 Claude Desktop 上使用

将此添加到您的 `claude_desktop_config.json`：

```json
{
  "mcpServers": {
    "mcp-server-firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY_HERE",

        "FIRECRAWL_RETRY_MAX_ATTEMPTS": "5",
        "FIRECRAWL_RETRY_INITIAL_DELAY": "2000",
        "FIRECRAWL_RETRY_MAX_DELAY": "30000",
        "FIRECRAWL_RETRY_BACKOFF_FACTOR": "3",

        "FIRECRAWL_CREDIT_WARNING_THRESHOLD": "2000",
        "FIRECRAWL_CREDIT_CRITICAL_THRESHOLD": "500"
      }
    }
  }
}
```

### 系统配置

服务器包含几个可通过环境变量设置的可配置参数。如果未配置，这里是默认值：

```typescript
const CONFIG = {
  retry: {
    maxAttempts: 3, // 速率限制请求的重试次数
    initialDelay: 1000, // 首次重试前的初始延迟（毫秒）
    maxDelay: 10000, // 重试之间的最大延迟（毫秒）
    backoffFactor: 2, // 指数退避乘数
  },
  credit: {
    warningThreshold: 1000, // 当信用使用达到此水平时发出警告
    criticalThreshold: 100, // 当信用使用达到此水平时发出临界警报
  },
};
```

这些配置控制：

1. **重试行为**

   - 自动重试由于速率限制而失败的请求
   - 使用指数退避以避免压垮 API
   - 示例：使用默认设置，重试将尝试在：
     - 第一次重试：1 秒延迟
     - 第二次重试：2 秒延迟
     - 第三次重试：4 秒延迟（上限为 maxDelay）

2. **信用使用监控**
   - 跟踪云 API 使用的信用消耗
   - 在指定阈值处提供警告
   - 帮助防止意外的服务中断
   - 示例：使用默认设置：
     - 在剩余 1000 信用时警告
     - 在剩余 100 信用时发出临界警报

### 速率限制和批量处理

服务器利用 FireCrawl 的内置速率限制和批量处理功能：

- 使用指数退避自动处理速率限制
- 高效的并行处理批量操作
- 智能请求排队和节流
- 自动重试临时错误

## 可用工具

### 1. 抓取工具 (`firecrawl_scrape`)

使用高级选项从单个 URL 抓取内容。

```json
{
  "name": "firecrawl_scrape",
  "arguments": {
    "url": "https://example.com",
    "formats": ["markdown"],
    "onlyMainContent": true,
    "waitFor": 1000,
    "timeout": 30000,
    "mobile": false,
    "includeTags": ["article", "main"],
    "excludeTags": ["nav", "footer"],
    "skipTlsVerification": false
  }
}
```

### 2. 批量抓取工具 (`firecrawl_batch_scrape`)

使用内置速率限制和并行处理高效抓取多个 URL。

```json
{
  "name": "firecrawl_batch_scrape",
  "arguments": {
    "urls": ["https://example1.com", "https://example2.com"],
    "options": {
      "formats": ["markdown"],
      "onlyMainContent": true
    }
  }
}
```

响应包括用于状态检查的操作 ID：

```json
{
  "content": [
    {
      "type": "text",
      "text": "Batch operation queued with ID: batch_1. Use firecrawl_check_batch_status to check progress."
    }
  ],
  "isError": false
}
```

### 3. 检查批量状态 (`firecrawl_check_batch_status`)

检查批量操作的状态。

```json
{
  "name": "firecrawl_check_batch_status",
  "arguments": {
    "id": "batch_1"
  }
}
```

### 4. 搜索工具 (`firecrawl_search`)

搜索网页并可选地从搜索结果中提取内容。

```json
{
  "name": "firecrawl_search",
  "arguments": {
    "query": "your search query",
    "limit": 5,
    "lang": "en",
    "country": "us",
    "scrapeOptions": {
      "formats": ["markdown"],
      "onlyMainContent": true
    }
  }
}
```

### 5. 爬取工具 (`firecrawl_crawl`)

使用高级选项启动异步爬取。

```json
{
  "name": "firecrawl_crawl",
  "arguments": {
    "url": "https://example.com",
    "maxDepth": 2,
    "limit": 100,
    "allowExternalLinks": false,
    "deduplicateSimilarURLs": true
  }
}
```

### 6. 提取工具 (`firecrawl_extract`)

使用 LLM 功能从网页中提取结构化信息。支持云 AI 和自托管 LLM 提取。

```json
{
  "name": "firecrawl_extract",
  "arguments": {
    "urls": ["https://example.com/page1", "https://example.com/page2"],
    "prompt": "Extract product information including name, price, and description",
    "systemPrompt": "You are a helpful assistant that extracts product information",
    "schema": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "price": { "type": "number" },
        "description": { "type": "string" }
      },
      "required": ["name", "price"]
    },
    "allowExternalLinks": false,
    "enableWebSearch": false,
    "includeSubdomains": false
  }
}
```

示例响应：

```json
{
  "content": [
    {
      "type": "text",
      "text": {
        "name": "Example Product",
        "price": 99.99,
        "description": "This is an example product description"
      }
    }
  ],
  "isError": false
}
```

#### 提取工具选项：

- `urls`：要提取信息的 URL 数组
- `prompt`：LLM 提取的自定义提示
- `systemPrompt`：指导 LLM 的系统提示
- `schema`：结构化数据提取的 JSON 模式
- `allowExternalLinks`：允许从外部链接提取
- `enableWebSearch`：启用网页搜索以获取额外上下文
- `includeSubdomains`：在提取中包含子域

使用自托管实例时，提取将使用您配置的 LLM。对于云 API，它使用 FireCrawl 的托管 LLM 服务。

## 日志系统

服务器包含全面的日志记录：

- 操作状态和进度
- 性能指标
- 信用使用监控
- 速率限制跟踪
- 错误情况

示例日志消息：

```
[INFO] FireCrawl MCP Server initialized successfully
[INFO] Starting scrape for URL: https://example.com
[INFO] Batch operation queued with ID: batch_1
[WARNING] Credit usage has reached warning threshold
[ERROR] Rate limit exceeded, retrying in 2s...
```

## 错误处理

服务器提供强大的错误处理：

- 自动重试临时错误
- 使用退避处理速率限制
- 详细的错误消息
- 信用使用警告
- 网络弹性

示例错误响应：

```json
{
  "content": [
    {
      "type": "text",
      "text": "Error: Rate limit exceeded. Retrying in 2 seconds..."
    }
  ],
  "isError": true
}
```

## 开发

```bash
# 安装依赖
npm install

# 构建
npm run build

# 运行测试
npm test
```

### 贡献

1. Fork 仓库
2. 创建您的功能分支
3. 运行测试：`npm test`
4. 提交拉取请求

## 许可证

MIT 许可证 - 详见 LICENSE 文件

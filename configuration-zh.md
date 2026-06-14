---

## layout: default
title: 配置指南

# 配置指南

Horizon 通过两个文件进行配置：`.env` 用于存放 API 密钥，`data/config.json` 用于配置信息源、AI 提供商和过滤选项。

## AI 提供商

配置用于评分和摘要生成的 AI 模型。

`api_key_env` 始终是环境变量名称，而不是 API 密钥本身。
请将密钥保存在 `.env` 或 shell 环境中，然后让 `api_key_env` 指向
该变量：

```bash
OPENAI_API_KEY=sk-your-key
GOOGLE_API_KEY=your-gemini-key
```

Horizon 启动时，环境变量优先，因为
`data/config.json` 不会存储密钥。在本地 VS Code 中运行时，请在仓库根目录创建
`.env`，并从同一根目录启动 Horizon。

常用 API 密钥环境变量名称：


| 提供商           | `api_key_env` 值        |
| ------------- | ---------------------- |
| Anthropic     | `ANTHROPIC_API_KEY`    |
| OpenAI        | `OPENAI_API_KEY`       |
| Azure OpenAI  | `AZURE_OPENAI_API_KEY` |
| Gemini        | `GOOGLE_API_KEY`       |
| MiniMax       | `MINIMAX_API_KEY`      |
| 阿里云 DashScope | `DASHSCOPE_API_KEY`    |
| 豆包            | `DOUBAO_API_KEY`       |
| DeepSeek      | `DEEPSEEK_API_KEY`     |


**Anthropic Claude**：

```json
{
  "ai": {
    "provider": "anthropic",
    "model": "claude-sonnet-4.5-20250929",
    "api_key_env": "ANTHROPIC_API_KEY",
    "throttle_sec": 0
  }
}
```

**OpenAI**：

```json
{
  "ai": {
    "provider": "openai",
    "model": "gpt-4",
    "api_key_env": "OPENAI_API_KEY",
    "throttle_sec": 0
  }
}
```

**Gemini**：

```json
{
  "ai": {
    "provider": "gemini",
    "model": "gemini-2.0-flash",
    "api_key_env": "GOOGLE_API_KEY",
    "throttle_sec": 0
  }
}
```

**Azure OpenAI**：

```json
{
  "ai": {
    "provider": "azure",
    "model": "gpt-4o-production",
    "api_key_env": "AZURE_OPENAI_API_KEY",
    "azure_endpoint_env": "AZURE_OPENAI_ENDPOINT",
    "api_version": "2024-10-21",
    "throttle_sec": 0
  }
}
```

请在 `.env` 中设置 `AZURE_OPENAI_API_KEY` 和 `AZURE_OPENAI_ENDPOINT`。`model` 字段应填写 Azure 部署名称，而不仅仅是基础模型系列名称。

**MiniMax**：

```json
{
  "ai": {
    "provider": "minimax",
    "model": "MiniMax-M3",
    "api_key_env": "MINIMAX_API_KEY",
    "throttle_sec": 0
  }
}
```

可用模型：`MiniMax-M3`、`MiniMax-M2.7`、`MiniMax-M2.7-highspeed`

**阿里云 DashScope**（OpenAI 兼容）：

```json
{
  "ai": {
    "provider": "ali",
    "model": "qwen-plus",
    "api_key_env": "DASHSCOPE_API_KEY",
    "throttle_sec": 0
  }
}
```

使用 [DashScope 兼容模式](https://help.aliyun.com/zh/dashscope/developer-reference/use-dashscope-by-calling-openai-api) 端点。在 `.env` 中设置 `DASHSCOPE_API_KEY`。可选：设置 `base_url` 以覆盖默认值 `https://dashscope.aliyuncs.com/compatible-mode/v1`。

### AI 限流

如果模型有严格的每分钟请求上限，可以在 `data/config.json` 中降低评分速度：

```json
{
  "ai": {
    "throttle_sec": 4.5
  }
}
```

- `throttle_sec`：每条内容评分之间的暂停时间（秒）。默认为 `0`。
- 对于免费套餐（约 15 次/分钟）的模型，`4.5` 是一个合理的起始值。
- 如果吞吐量充足且希望最快速度，可将其设回 `0`。

### AI 并发

默认情况下，AI 评分和 enrichment 一次只处理一条内容。如果 API 端点支持并发请求，可以提高吞吐量：

```json
{
  "ai": {
    "analysis_concurrency": 4,
    "enrichment_concurrency": 2
  }
}
```

- `analysis_concurrency`：并行评分的条目数。默认为 `1`。
- `enrichment_concurrency`：并行 enrichment 的高分条目数。默认为 `1`。
- 两个值的最小值均为 `1`。
- 保留每条内容现有的重试行为。
- 无论并发度如何，结果顺序都会保持不变。
- 如果同时使用 `throttle_sec`，每个并发任务在完成一条内容后会独立休眠。

**自定义 Base URL**（用于代理）：

```json
{
  "ai": {
    "provider": "anthropic",
    "base_url": "https://your-proxy.com/v1",
    ...
  }
}
```

对于 OpenAI 兼容网关，Horizon 默认会发送 `temperature` 参数。如果较新的推理型模型因该参数报错（例如 `temperature is deprecated for this model`），Horizon 会重试一次（不带该参数），并记住该能力以供后续请求使用。

## 信息源

所有信息源均在 `config.json` 顶层的 `sources` 键下配置。

### GitHub

```json
{
  "sources": {
    "github": [
      {
        "type": "user_events",
        "username": "gvanrossum",
        "enabled": true
      },
      {
        "type": "repo_releases",
        "owner": "python",
        "repo": "cpython",
        "enabled": true
      }
    ]
  }
}
```

### Hacker News

```json
{
  "sources": {
    "hackernews": {
      "enabled": true,
      "fetch_top_stories": 30,
      "min_score": 100
    }
  }
}
```

### RSS 订阅

```json
{
  "sources": {
    "rss": [
      {
        "name": "Blog Name",
        "url": "https://example.com/feed.xml",
        "enabled": true,
        "category": "ai-ml"
      }
    ]
  }
}
```

### Reddit

```json
{
  "sources": {
    "reddit": {
      "enabled": true,
      "fetch_comments": 5,
      "subreddits": [
        {
          "subreddit": "MachineLearning",
          "sort": "hot",
          "fetch_limit": 25,
          "min_score": 10
        }
      ],
      "users": [
        {
          "username": "spez",
          "sort": "new",
          "fetch_limit": 10
        }
      ]
    }
  }
}
```

### Telegram

Telegram 抓取使用 `https://t.me/s/<channel>` 的公开网页预览，无需 API 密钥。仅支持公开频道。

```json
{
  "sources": {
    "telegram": {
      "enabled": true,
      "channels": [
        {
          "channel": "zaihuapd",
          "enabled": true,
          "fetch_limit": 20
        }
      ]
    }
  }
}
```

- `enabled` — 全局启用或禁用 Telegram 抓取
- `channels` — 要监控的公开 Telegram 频道列表
- `channel` — Telegram 频道用户名，不含 `@` 或完整的 `https://t.me/` URL
- `fetch_limit` — 每次运行每个频道检查的最新消息数量上限（默认：`20`）

### Twitter

需要 [Apify](https://apify.com) 账户。在 `.env` 文件中设置 `APIFY_TOKEN`。免费套餐包含每月 $5 额度，大约可抓取 20,000 条推文。

```json
{
  "sources": {
    "twitter": {
      "enabled": true,
      "users": ["karpathy", "ylecun"],
      "fetch_limit": 10,
      "fetch_reply_text": false,
      "max_replies_per_tweet": 3,
      "max_tweets_to_expand": 10,
      "reply_min_likes": 5
    }
  }
}
```

- `users` — 要监控的 Twitter 用户名，不含 `@` 前缀
- `fetch_limit` — 每次运行抓取的最大推文数（所有用户合计；因 actor 限制，最小为 100）
- `fetch_reply_text` — 为 `true` 时，为重要推文抓取实际回复正文，并追加到 `--- Top Comments ---` 下，以便 AI 纳入社区讨论。默认关闭。
- `max_replies_per_tweet` — 每条推文追加的最大回复行数（默认：3）
- `max_tweets_to_expand` — 每次运行展开回复的推文数量上限，用于控制 Apify 额度消耗（默认：10）
- `reply_min_likes` — 仅包含点赞数至少达到此值的回复（默认：0）

抓取器默认使用 `altimis/scweet` actor。如有需要，可通过 `actor_id` 覆盖。

### OpenBB 财经新闻

当你希望通过一个 SDK 从 yfinance、Benzinga、FMP、Intrinio、Tiingo、SEC 或美联储等提供商获取股票或宏观新闻时，OpenBB 很有用。

启用该信息源前，请先安装可选依赖：

```bash
uv sync --extra openbb
```

如果平台在构建传递依赖时遇到困难，建议使用：

```bash
uv pip install --only-binary=:all: openbb openbb-benzinga
```

```json
{
  "sources": {
    "openbb": {
      "enabled": true,
      "watchlists": [
        {
          "name": "megacaps",
          "enabled": true,
          "provider": "yfinance",
          "fetch_limit": 20,
          "category": "equities",
          "symbols": ["AAPL", "MSFT", "NVDA", "GOOGL", "AMZN", "META", "TSLA"]
        }
      ]
    }
  }
}
```

- `enabled` — 全局启用或禁用 OpenBB 信息源
- `watchlists` — 命名股票组列表；每个 watchlist 每次运行对应一次 `news.company()` 调用
- `name` — 在 Horizon 元数据和筛选明细中显示的标签
- `provider` — OpenBB 提供商名称，如 `yfinance` 或 `benzinga`
- `fetch_limit` — 该 watchlist 请求的最大新闻条数
- `category` — 可选标签，存储在抓取到的条目上
- `symbols` — 要一起抓取的股票代码；按提供商分组以保持请求高效

OpenBB 提供商凭据由 OpenBB SDK 自行处理，使用其环境变量或用户设置。Horizon 不会通过 `data/config.json` 传递这些密钥。

### OSS Insight（GitHub 热门仓库）

从 [OSS Insight](https://ossinsight.io) 公开 API 拉取 star 增长最多的仓库，该 API 聚合 GitHub WatchEvents。适合发现当前正在快速涨星的仓库，无需抓取 GitHub Trending 或查询 BigQuery。

```json
{
  "sources": {
    "ossinsight": {
      "enabled": true,
      "period": "past_24_hours",
      "languages": ["All", "Python", "TypeScript"],
      "keywords": [],
      "min_stars": 10,
      "max_items": 30
    }
  }
}
```

- `period` — star 增长排名的时间窗口。支持：`past_24_hours`、`past_28_days`。（`past_7_days` 目前在上游已损坏。）
- `languages` — 要查询的主要语言分类。使用 `"All"` 获取完整排名，或使用任意 GitHub 语言标签，如 `"Python"`、`"TypeScript"`、`"Rust"`、`"Jupyter Notebook"`。抓取器会为每种语言发起一次请求并合并结果。
- `keywords` — 可选的不区分大小写子字符串，匹配 `description`、`collection_names` 和 `repo_name`。仅包含至少一个关键词的仓库会通过。留空则抓取所有 trending 仓库。
- `min_stars` — 过滤掉在该周期内 star 增长少于此值的仓库。
- `max_items` — 合并并按 `stars_gained` 降序排序后的最终数量上限。

无需 API 密钥。

## 过滤

内容评分为 0-10：

- **9-10**：突破性 — 重大突破、范式转变
- **7-8**：高价值 — 重要进展、深度技术内容
- **5-6**：值得关注 — 值得了解但不紧急
- **3-4**：低优先级 — 通用或常规内容
- **0-2**：噪音 — 垃圾信息、离题或琐碎内容

```json
{
  "filtering": {
    "ai_score_threshold": 7.0,
    "time_window_hours": 24,
    "max_items": 20,
    "category_groups": {
      "ai": {
        "name": "AI / Machine Learning",
        "limit": 5,
        "categories": ["ai-news", "ai-tools", "machine-learning", "llm"]
      },
      "finance": {
        "name": "Finance",
        "limit": 5,
        "categories": ["finance", "equities", "crypto"]
      }
    },
    "default_group": "other",
    "default_group_limit": 3
  }
}
```

- `ai_score_threshold`：仅包含评分 >= 此值的内容
- `time_window_hours`：抓取最近 N 小时的内容
- `max_items`：应用所有分组限额后的可选最终上限
- `category_groups`：可选的配额分组映射。每个分组需要正数
`limit` 和非空的 `categories` 列表。组内条目按
AI 评分从高到低保留。
- `category_groups.*.name`：运行日志中使用的可选显示名称
- `default_group`：类别不匹配任何
已配置分组的条目所属的分组键。默认为 `other`。
- `default_group_limit`：未匹配条目的可选正数限额。若省略，
未匹配条目不受限，仅受 `max_items` 约束。

均衡摘要过滤在 AI 评分阈值过滤和主题
去重之后、enrichment 之前运行。这样可以减少 enrichment 调用，仅处理
可能出现在最终摘要中的条目。

分组匹配使用存储在 `ContentItem.metadata.category` 中的来源类别。
RSS 信息源通过 `sources.rss[].category` 暴露该字段，OpenBB watchlist
通过 `sources.openbb.watchlists[].category` 暴露。没有类别的信息源进入
默认分组。

如果同一类别出现在多个分组中，Horizon 会记录警告并使用
配置顺序中的第一个分组。若同时省略 `category_groups` 和
`max_items`，则保留之前的过滤行为。

## 环境变量替换

`data/config.json` 中的任意字符串值均支持 `${VAR_NAME}` 语法。变量在运行时从环境（包括从 `.env` 加载的值）中展开。这样可以将密钥、租户特定端点和私有 URL 保留在已提交的 JSON 文件之外。

示例：

```json
{
  "ai": {
    "base_url": "${HORIZON_AI_BASE_URL}"
  },
  "sources": {
    "rss": [
      {
        "name": "LWN.net",
        "url": "https://lwn.net/headlines/full_text?key=${LWN_KEY}",
        "enabled": true
      }
    ]
  },
  "webhook": {
    "url_env": "HORIZON_WEBHOOK_URL",
    "headers": "Authorization: Bearer ${HORIZON_WEBHOOK_TOKEN}"
  }
}
```

- `${NAME}` 仅在 `NAME` 是有效标识符（如 `LWN_KEY` 或 `HORIZON_AI_BASE_URL`）时才会被替换。
- 未设置的变量会保留为 `${NAME}`，而不会变成空字符串，以便配置错误能在下游明显暴露。
- 展开会递归遍历 dict、list 和 tuple；非字符串值保持不变。

## 邮件订阅

邮件投递是可选的，除非 `email.enabled` 为 `true`，否则不会启用。Horizon 使用 SMTP 发送每日摘要，使用 IMAP 检查订阅/退订请求。

```json
{
  "email": {
    "enabled": true,
    "smtp_server": "smtp.qq.com",
    "smtp_port": 465,
    "smtp_username": null,
    "imap_enabled": true,
    "imap_server": "imap.qq.com",
    "imap_port": 993,
    "email_address": "xxx@qq.com",
    "password_env": "EMAIL_PASSWORD",
    "sender_name": "Horizon Daily",
    "subscribe_keyword": "SUBSCRIBE",
    "unsubscribe_keyword": "UNSUBSCRIBE"
  }
}
```

- `enabled`：开启或关闭邮件订阅处理和每日邮件投递。
- `smtp_server` / `smtp_port`：用于发送邮件的 SMTP 服务器。
- `smtp_username`：可选的 SMTP 登录用户名。若省略，Horizon 使用 `email_address`。
- `imap_enabled`：开启或关闭 IMAP 订阅/退订检查。对于仅发送的 SMTP 提供商，设为 `false`。
- `imap_server` / `imap_port`：当 `imap_enabled` 为 `true` 时，用于扫描订阅请求的 IMAP 服务器。
- `email_address`：发件账户和检查订阅请求的邮箱。
- `password_env`：包含邮件密码或应用密码的环境变量。默认为 `EMAIL_PASSWORD`。
- `sender_name`：发送邮件中显示的名称。
- `subscribe_keyword` / `unsubscribe_keyword`：Horizon 在收到的邮件主题中查找的关键词。

Resend SMTP 示例：

```json
{
  "email": {
    "enabled": true,
    "smtp_server": "smtp.resend.com",
    "smtp_port": 465,
    "smtp_username": "resend",
    "password_env": "RESEND_API_KEY",
    "imap_enabled": false,
    "imap_server": "",
    "imap_port": 993,
    "email_address": "noreply@example.com",
    "sender_name": "Horizon Daily"
  }
}
```

在 `.env` 中设置 `RESEND_API_KEY`。收件人从 `data/subscribers.json` 加载。

## Webhook 通知

Webhook 通知是可选的，除非 `webhook.enabled` 为 `true`，否则不会启用。Horizon 可在流水线成功或失败时调用飞书/Lark、钉钉、Slack、Discord 或任意自定义 webhook 端点。

```json
{
  "webhook": {
    "enabled": true,
    "url_env": "HORIZON_WEBHOOK_URL",
    "delivery": "summary",
    "overview_position": "first",
    "platform": "generic",
    "layout": "markdown",
    "fallback_layout": "markdown",
    "languages": null,
    "request_body": {
      "text": "#{message_title}\n#{summary}"
    },
    "headers": ""
  }
}
```

- `enabled`：开启或关闭 webhook 投递。默认为 `false`。
- `url_env`：包含 webhook URL 的环境变量。例如，在 `.env` 中设置 `HORIZON_WEBHOOK_URL=https://...`。
- `delivery`：控制消息发送方式。使用 `summary` 发送一条完整消息，或使用 `summary_and_items` 发送一条概览消息，再为每个选中条目各发一条消息。
- `overview_position`：在 `summary_and_items` 模式下控制概览的发送位置。使用 `first` 为传统顺序，或使用 `last` 先发送条目详情（倒序），使概览成为最新的聊天消息。
- `platform`：可选的 webhook 平台提示。默认使用 `generic`，或使用 `feishu` / `lark` 启用平台特定的卡片渲染。
- `layout`：控制消息布局。使用 `markdown` 进行模板化 Markdown 投递，或在 `platform: "feishu"` / `"lark"` 时使用 `collapsible` 发送一条飞书 Card JSON 2.0 消息，每个条目在折叠面板中。
- `fallback_layout`：不支持的平台/布局组合时的备用布局。当前安全的备用值为 `markdown`。
- `languages`：可选的 webhook 专用语言过滤器。使用 `["zh"]` 或 `["en"]` 仅发送选定语言；使用 `null` 或省略以发送所有已配置的 `ai.languages`。
- `request_body`：可选的请求体。若为空，Horizon 发送 `GET` 请求。若提供，Horizon 发送 `POST` 请求。
- `headers`：可选的自定义请求头，每行一个 `Key: Value` 对。

当 `request_body` 是 JSON 对象或数组时，Horizon 渲染占位符并将其序列化为 JSON。当它是字符串时，Horizon 直接渲染，并在渲染后的字符串是有效 JSON 时自动识别。

### 投递模式与布局

`delivery` 控制 Horizon 发送多少条 webhook 消息：

- `summary`：发送一条包含完整每日摘要的消息。简单，但某些聊天平台可能拒绝过长的消息。
- `summary_and_items`：发送一条概览消息，再为每个选中条目各发一条消息。在每条条目消息中，`#{summary}` 仅包含该条目的 Markdown 正文。适用于会拒绝或截断长消息的平台。

`layout` 控制每条消息的渲染方式：

- `markdown`：对每条消息使用你的 `request_body` 模板。这是默认值，适用于通用 webhook、钉钉、Slack、Discord、飞书和 Lark。
- `collapsible`：当前支持 `platform: "feishu"` 或 `"lark"`。Horizon 忽略 `request_body`，构建一条飞书/Lark Card JSON 2.0 消息，每个条目在折叠面板中。

对于没有平台特定布局的平台，保持 `layout: "markdown"`，并通过 `delivery` 选择消息数量。

`summary_and_items` Markdown 投递配置示例：

```json
{
  "webhook": {
    "enabled": true,
    "url_env": "HORIZON_WEBHOOK_URL",
    "delivery": "summary_and_items",
    "overview_position": "last",
    "platform": "generic",
    "layout": "markdown",
    "request_body": {
      "text": "#{message_title}\n\n#{summary?limit=3000&split=---}"
    }
  }
}
```

使用 `summary_and_items` 时，Horizon 发送一条概览加每个选中条目各一条消息。`overview_position: "last"` 先发送条目消息，使概览成为最新的聊天消息；省略或设为 `"first"` 则先发送概览。

### Webhook 模板

可用变量：


| 变量                   | 说明                                                        |
| -------------------- | --------------------------------------------------------- |
| `#{date}`            | 报告日期，例如 `2026-04-24`                                      |
| `#{language}`        | 语言代码，如 `en` 或 `zh`                                        |
| `#{important_items}` | 通过评分阈值的条目数                                                |
| `#{all_items}`       | 抓取到的总条目数                                                  |
| `#{result}`          | `success` 或 `failed`                                      |
| `#{timestamp}`       | Unix 时间戳                                                  |
| `#{message_title}`   | 消息标题，如每日标题、概览标题或条目标题                                      |
| `#{message_kind}`    | 消息类型：`summary`、`overview`、`item`、`failure` 或 `manual`     |
| `#{summary}`         | 消息 Markdown。在 `summary_and_items` 模式下，这是概览或单条条目正文，取决于消息类型 |


当 `delivery` 为 `summary_and_items` 时，条目消息还包含：


| 变量              | 说明          |
| --------------- | ----------- |
| `#{item_index}` | 从 1 开始的条目序号 |
| `#{item_count}` | 条目消息总数      |
| `#{item_title}` | 当前条目标题      |
| `#{item_url}`   | 当前条目 URL    |
| `#{item_score}` | 当前条目 AI 评分  |


对于 webhook 投递，Horizon 会将 `#{summary}` 中的 HTML 折叠块（如 `<details><summary>...</summary>`）展平为纯 Markdown 链接列表。这使生成的摘要更易于在聊天产品中渲染。保存的 Markdown 文件、GitHub Pages 和邮件内容不受影响。

使用 `#{key?limit=N&split=DELIM}` 通过在 `DELIM` 处分割并保留片段直到总字符数达到 `N` 来截断长值。

```text
#{summary?limit=3000&split=---}
```

### 钉钉

在钉钉中，创建自定义群机器人并使用自定义关键词（如 `Horizon`）。关键词必须出现在消息正文中。

```json
{
  "msgtype": "markdown",
  "markdown": {
    "title": "Horizon #{date} Daily",
    "text": "Horizon result: #{result}\n\nHorizon important items: #{important_items}/#{all_items}\n\n#{summary}"
  }
}
```

### 飞书 / Lark

在飞书或 Lark 中，创建自定义群机器人并使用自定义关键词（如 `Horizon`）。关键词必须出现在消息正文中。

使用 Card JSON 2.0 进行 Markdown 渲染。卡片必须包含 `"schema": "2.0"`，并将富文本 Markdown 组件放在 `card.body.elements` 下。

要在保持群聊紧凑的同时，仍允许读者在飞书内浏览完整简报，可使用折叠布局：

```json
{
  "webhook": {
    "enabled": true,
    "url_env": "HORIZON_WEBHOOK_URL",
    "platform": "feishu",
    "layout": "collapsible",
    "fallback_layout": "markdown",
    "languages": ["zh"]
  }
}
```

使用此布局时，Horizon 发送一条交互式卡片，包含概览和每个选中条目的一个折叠面板。每个面板可在飞书中展开以阅读完整条目详情。常规的 `request_body` 模板对此渲染卡片会被忽略。

```json
{
  "msg_type": "interactive",
  "card": {
    "schema": "2.0",
    "config": {
      "wide_screen_mode": true
    },
    "header": {
      "title": {
        "tag": "plain_text",
        "content": "#{message_title}"
      },
      "template": "blue"
    },
    "body": {
      "elements": [
        {
          "tag": "markdown",
          "content": "Horizon result: #{result}\nHorizon important items: #{important_items}/#{all_items}"
        },
        {
          "tag": "hr"
        },
        {
          "tag": "markdown",
          "content": "#{summary}"
        }
      ]
    }
  }
}
```

## 静态站点

Horizon 将生成的摘要写入 `data/summaries/`，并将可发布的 Markdown 复制到 `docs/` 以供 GitHub Pages 站点使用。仓库包含现成的 workflow：`.github/workflows/daily-summary.yml`。

要使用 GitHub Pages，请为仓库启用 Pages，并运行定时 workflow 或手动触发。生成的站点从 `docs/` 目录构建。

## MCP 服务器

Horizon 包含一个 MCP 服务器，供 AI 助手和 MCP 兼容客户端使用。

```bash
uv run horizon-mcp
```

可用工具包括 `hz_validate_config`、`hz_fetch_items`、`hz_score_items`、`hz_filter_items`、`hz_enrich_items`、`hz_generate_summary` 和 `hz_run_pipeline`。

完整工具参考见 `[src/mcp/README.md](../src/mcp/README.md)`，客户端配置见 `[src/mcp/integration.md](../src/mcp/integration.md)`。
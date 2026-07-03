# lucen.cc 大模型中转站社区

`lucen.cc` 是面向开发者的大模型 API 中转与统一网关服务，基于 Sub2API 体系做了 Lucen 自有的账号池、模型路由、用量统计和计费归因等定制能力。

本仓库是 lucen.cc 的公开社区仓库，用于沉淀接入说明、常见问题、问题反馈、用户交流讨论和社区建议。后端生产环境的密钥、数据库、Redis、服务器密码、JWT、TOTP、代理密钥等敏感信息不会放在这里。

## 项目定位

- 提供 OpenAI 兼容的 API 调用入口
- 提供 Gemini 风格的模型调用入口
- 支持多模型、多上游账号池的统一路由
- 支持 API Key 管理、用量统计、余额/套餐等业务能力
- 为 lucen.cc 用户提供接入文档、问题反馈、用户交流讨论和变更说明

## 服务地址

主站：

- <https://lucen.cc>

健康检查：

- <https://lucen.cc/health>

常用 API Base URL：

| 类型 | Base URL |
| --- | --- |
| OpenAI 兼容接口 | `https://lucen.cc/v1` |
| OpenAI 兼容备用前缀 | `https://lucen.cc/openai/v1` |
| Key 管理相关接口 | `https://lucen.cc/keys/v1` |
| Gemini v1beta 风格接口 | `https://lucen.cc/v1beta` |
| Gemini v1 风格接口 | `https://lucen.cc/v1` |

实际可用模型请以控制台展示或 `/v1/models` 返回结果为准。

## 快速开始

### OpenAI SDK

把 SDK 的 `base_url` 改为 `https://lucen.cc/v1`，并使用你在 lucen.cc 控制台创建的 API Key。

```python
from openai import OpenAI

client = OpenAI(
    api_key="你的 lucen.cc API Key",
    base_url="https://lucen.cc/v1",
)

response = client.chat.completions.create(
    model="gpt-5.4",
    messages=[
        {"role": "user", "content": "用一句话介绍 lucen.cc"},
    ],
)

print(response.choices[0].message.content)
```

### curl 调用

```bash
curl https://lucen.cc/v1/chat/completions \
  -H "Authorization: Bearer 你的 lucen.cc API Key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.4",
    "messages": [
      { "role": "user", "content": "你好，介绍一下你自己" }
    ]
  }'
```

### 查看模型列表

```bash
curl https://lucen.cc/v1/models \
  -H "Authorization: Bearer 你的 lucen.cc API Key"
```

### Gemini 风格接口

```bash
curl "https://lucen.cc/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "Authorization: Bearer 你的 lucen.cc API Key" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [
      {
        "parts": [
          { "text": "你好，请用中文回答" }
        ]
      }
    ]
  }'
```

## 支持的接口形态

当前公开入口覆盖以下常见形态：

- `https://lucen.cc/v1/*`
- `https://lucen.cc/openai/v1/*`
- `https://lucen.cc/keys/v1/*`
- `https://lucen.cc/chat/completions`
- `https://lucen.cc/images/generations`
- `https://lucen.cc/v1beta/models/{model}:generateContent`
- `https://lucen.cc/v1beta/models/{model}:streamGenerateContent?alt=sse`
- `https://lucen.cc/v1beta/models/{model}:countTokens`
- `https://lucen.cc/v1/models/{model}:generateContent`
- `https://lucen.cc/v1/models/{model}:streamGenerateContent?alt=sse`
- `https://lucen.cc/v1/models/{model}:countTokens`

## Lucen 的路由与计费概念

Lucen 的 API Key 会区分两个概念：

- 计费来源：决定请求消耗哪个套餐、余额或计费规则
- 路由账号池：决定请求实际走哪个上游平台、模型映射和账号池

这意味着用户可以在一个计费来源下，把不同 Key 路由到 OpenAI、Claude、Gemini 或其他模型池。模型是否可用、请求是否成功，最终以当前账号池配置和上游状态为准。

## 当前分组与倍率

分组和倍率会随后台运营调整，以下为当前公开接入口径，最终以 lucen.cc 控制台实时展示为准。

倍率含义：

- 余额倍率：使用余额计费时，请求按该分组倍率折算消耗
- 套餐倍率：使用套餐权益时，请求按该分组倍率计入套餐额度

常用路由分组：

| 分组 | 平台 | 余额倍率 | 套餐倍率 | 适用场景 |
| --- | --- | ---: | ---: | --- |
| `plus号池-codex官转` | OpenAI | 1.3x | 4x | Plus / Codex 官转池，适合作为通用 OpenAI 兼容入口 |
| `plus号池-codex` | OpenAI | 1.3x | 4x | 稳定可用的 Codex/Plus 池 |
| `codex-pro-new` | OpenAI | 2x | 6x | Pro/Codex 高阶池 |
| `gpt生图image2` | OpenAI | 1.3x | 4x | GPT 生图专用池，当前生图价格配置为 1k/2k/4k 均 0.05 |
| `cc逆向支持4.7支持1m上下文` | Anthropic | 4x | 13x | Claude Code / Claude 长上下文场景 |
| `反重力Claude code逆向` | Anthropic | 4x | 15x | Claude Code 逆向池 |
| `gemini+生图` | Gemini | 4x | 15x | Gemini 文本与生图场景，生图价格配置为 1k/2k/4k 均 2 |

历史兼容或特殊分组可能仍在后台存在，例如旧 OpenAI 池、特价 Claude 池、待下线分组等。新建 Key 时建议优先选择控制台推荐的分组。

## 套餐支持

Lucen 同时支持余额计费和套餐权益：

- 余额计费：按所选路由分组的余额倍率消耗余额
- 套餐权益：按所选计费来源的日/周/月额度限制使用，并按路由分组的套餐倍率折算
- 混合路由：一个套餐或余额来源可以绑定到不同路由账号池，例如 OpenAI 计费来源 + Claude/Gemini 路由池

当前订阅额度分组：

| 套餐分组 | 日额度 | 周额度 | 月额度 | 说明 |
| --- | ---: | ---: | ---: | --- |
| `50刀订阅` | 50 | 350 | 1550 | 支持日卡/周卡/月卡 |
| `100刀订阅` | 100 | 700 | 3100 | 支持日卡/周卡/月卡 |
| `200刀订阅` | 200 | 1400 | 6200 | 支持日卡/周卡/月卡 |
| `300刀订阅` | 300 | 2100 | 9300 | 主要用于日卡/周卡，实际售卖以控制台为准 |
| `500刀订阅` | 500 | 3500 | 15500 | 主要用于日卡/周卡，实际售卖以控制台为准 |

套餐额度单位沿用后台美元额度口径，用于统一计算不同模型和不同账号池的消耗，并不等同于直接售卖价格。

## 常见问题

### 1. 这个仓库是 lucen.cc 后端源码吗？

不是。本仓库当前定位是 lucen.cc 大模型中转站的公开社区与接入说明仓库。生产后端、环境变量、数据库配置和服务器运维资料不会放在公开仓库里。

### 2. 为什么同一个模型名有时不可用？

模型可用性取决于当前套餐、Key 权限、路由账号池、上游账号状态和上游服务本身。如果遇到不可用，请优先确认模型列表、Key 权限和余额状态。

### 3. OpenAI SDK 应该怎么配置？

通常只需要改两项：

- `api_key`：填写 lucen.cc 生成的 API Key
- `base_url`：填写 `https://lucen.cc/v1`

### 4. 能不能把 API Key 提交到 Issue 或截图里？

不要。请不要在 Issue、截图、日志或公开聊天中暴露 API Key、账号密码、订单敏感信息和完整请求内容。

## 仓库使用方式

这个仓库主要用于公开协作，不处理需要暴露账号、订单、密钥或服务器信息的私密问题。

建议这样使用：

- Bug 反馈：接口报错、模型不可用、计费展示异常、文档描述不准
- 使用求助：SDK 配置、Base URL、模型名、流式输出、生图接口等接入问题
- 经验交流：不同客户端、不同 SDK、不同模型池的使用经验
- 功能建议：新模型、新接口、控制台体验、套餐展示、路由逻辑等建议
- 公告沉淀：服务变更、分组调整、接口兼容说明、已知问题说明

不建议公开提交：

- API Key、Access Token、账号密码、Cookie、JWT、TOTP、代理密钥
- 订单号、支付截图、实名信息、邮箱、手机号等个人信息
- 完整对话内容、业务数据、客户数据或其他敏感输入输出
- 服务器 IP、数据库连接、Redis 密码、`.env`、后台管理地址

## 问题反馈与用户交流

本仓库是 lucen.cc 用户交流和问题反馈的公开入口。欢迎通过 Issue 或 Discussion 交流：

- 接入问题
- 模型不可用或响应异常
- 文档错误
- SDK 示例需求
- 新模型或新接口建议
- 使用经验、配置经验和最佳实践
- 套餐、分组、倍率、路由逻辑相关疑问
- 社区公告、变更记录和后续规划讨论

提交问题时建议提供：

- 使用的接口路径
- 模型名
- SDK 或调用方式
- 错误码和错误信息
- 不包含密钥的最小复现请求

可以参考下面的反馈格式：

```markdown
## 问题类型

接入问题 / 模型不可用 / 计费疑问 / 文档错误 / 功能建议 / 其他

## 接口与模型

- 接口路径：
- Base URL：
- 模型名：
- SDK / 客户端：

## 现象

请描述实际返回、错误码、错误信息或异常表现。

## 期望

请描述你期望发生什么。

## 已做排查

- 是否确认 API Key 未过期：
- 是否确认余额或套餐仍可用：
- 是否确认模型名在控制台或 `/v1/models` 中存在：
- 是否已隐藏所有密钥、账号和个人信息：
```

## 排查前自检

遇到请求失败时，可以先按下面顺序确认：

1. API Key 是否正确，前后没有多余空格
2. Base URL 是否为 `https://lucen.cc/v1` 或文档中列出的兼容入口
3. 模型名是否在控制台或 `/v1/models` 中存在
4. 当前 Key 绑定的计费来源、路由账号池、余额或套餐是否可用
5. 请求体是否符合对应接口格式，例如 OpenAI 兼容接口和 Gemini 风格接口不要混用
6. 如果是流式输出，确认客户端支持 SSE/stream
7. 如果是生图接口，确认当前分组允许生图且选择了支持的模型

## 安全与合规提醒

使用 lucen.cc 时，请遵守所在地法律法规、上游模型服务条款和平台使用规范。不要提交违法、有害、侵权或包含敏感个人信息的内容。公开反馈时请主动脱敏。

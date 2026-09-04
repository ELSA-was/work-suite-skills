---
name: lingyi-mcp
description: 零一运营 MCP。用 open_lingyi_skill 按 slug 打开对应 Skill 的 MCP App 表单。
version: 1.1.0
author: 零一运营
---

# 零一运营 MCP

唯一工具：`open_lingyi_skill`。不要向用户要 API Key / `user_id`，也不要把凭证写进 tool 参数。

用户要使用某个 Skill 时，调用 `open_lingyi_skill(skill_id)` 打开 MCP App 表单，不要用纯文本代替表单。`skill_id` 必须是下表 slug。

打开表单后等用户在 MCP App 里填写并提交。提交后参数会以一条用户消息送回对话（含 `source: mcp_app`、`validated: true` 与参数 JSON）。收到后不要重复追问已提供的字段，也不要声称已经创建或完成任务。

## 工具

### open_lingyi_skill

打开该 Skill 的交互表单 MCP App。每个 slug 对应一份独立 HTML 表单。

| 参数 | 类型 | 必填 | 取值 | 默认 |
|------|------|------|------|------|
| `skill_id` | string | 是 | 下表五个 slug 之一 | 无 |

#### 成功

- `isError`: false
- 文本：`已打开 {Skill 中文名} 表单。请让用户在 MCP App 中填写参数。`
- `structuredContent.status`: `ready`
- 同时带上 `skill_id`、`name`、`estimate`、`form_uri`、`inputs`
- 宿主会打开 MCP App（`ui://lingyi-skill/app.html`）

#### 失败

- `isError`: true
- `structuredContent.status`: `validation_error`
- `errors`: 字符串数组。`skill_id` 无效时会列出当前支持的 slug

## 错误场景

| 场景 | 行为 |
|------|------|
| `skill_id` 不在清单 | 把工具返回的支持列表告诉用户，改用正确 slug 再调用 |
| 表单长时间未提交 | 继续等待；不要编造参数，不要假装任务已创建 |
| `share_url` 不是 `weixin.qq.com` / `channels.weixin.qq.com` 的 HTTPS 链接 | 表单会拒绝。引导用户换成合法视频号分享链接后再提交 |
| 表单 `mode` 为 `rewrite,eval` | 这是默认全流程，不是错误。先改写再评估，两次独立任务、独立扣点确认 |
| 用户只在对话里口述参数 | 仍先打开表单；以表单回传且 `validated: true` 的 JSON 为准 |

## 当前支持的 slug 与表单入参

### `lingyi-wx-video-decomposer-plus` 视频号爆款短视频拆解

| 参数 | 类型 | 必填 | 取值范围 | 默认 |
|------|------|------|----------|------|
| `share_url` | string | 是 | 仅 `https://weixin.qq.com` 或 `https://channels.weixin.qq.com` | 无 |
| `industry` | string | 否 | 如 food / 美妆 | 空 |
| `campaign_type` | string | 否 | 如 live_commerce | 空 |
| `account_size` | string | 否 | 如 1-100k | 空 |

### `lingyi-daily-hot-topic` 每日热点选题

| 参数 | 类型 | 必填 | 取值范围 | 默认 |
|------|------|------|----------|------|
| `industry` | string | 是 | 如：美妆 | 无 |
| `brand_keywords` | string | 否 | 逗号分隔，可留空 | 空 |
| `platforms` | string | 否 | `douyin` / `xiaohongshu` / `weibo` / `kuaishou` / `zhihu` | 空=抖音+小红书 |
| `goal` | string | 否 | 营销目标 | 种草 |
| `count` | number | 否 | 5–10 | 5 |
| `additional_requirements` | string | 否 | 额外要求 | 空 |

### `lingyi-copy-de-ai-human-eval` 文案去 AI 味 · 真人评估

| 参数 | 类型 | 必填 | 取值范围 | 默认 |
|------|------|------|----------|------|
| `mode` | string | 是 | `rewrite` / `eval`，可组合为 `rewrite,eval` | `rewrite,eval` |
| `text` | string | 是 | 改写填原文；评估填改写终稿 | 无 |
| `platform` | string | 否 | 发布平台 | `xhs` |
| `personas` | string | 否 | 仅 `eval`；留空用默认 5 类读者 | 空 |
| `comment_count` | number | 否 | 仅 `eval`；1–20 | 6 |

### `lingyi-content-quality-check` 爆款内容预检

| 参数 | 类型 | 必填 | 取值范围 | 默认 |
|------|------|------|----------|------|
| `text` | string | 是 | 待检文案，1–20000 字 | 无 |
| `modules` | string | 是 | `compliance` / `persona` / `burst`，可组合 | 三块全跑 |
| `platform` | string | 否 | 仅 `channels`（视频号）/ `mp`（公众号） | `channels` |
| `personas` | string | 否 | 仅 `persona`；`年龄段-性别-职业-消费层级`，逗号分隔，可留空 | 空 |
| `comment_count` | number | 否 | 仅 `persona`；1–20 | 6 |

### `lingyi-wx-account-quick-analysis` 微信视频号账号拆解

| 参数 | 类型 | 必填 | 取值范围 | 默认 |
|------|------|------|----------|------|
| `account_name` | string | 是 | 从微信视频号复制的准确昵称 | 无 |

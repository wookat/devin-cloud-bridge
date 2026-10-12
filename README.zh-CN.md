# devin-cloud-bridge

[English](./README.md) · **简体中文**

<p>
  <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-blue.svg">
  <img alt="MCP" src="https://img.shields.io/badge/MCP-compatible-green">
  <img alt="Devin API v3" src="https://img.shields.io/badge/Devin_API-v3-7c3aed">
  <img alt="PRs welcome" src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg">
</p>

一个 [Devin CLI](https://devin.ai) 插件，打通**本地会话 ↔ Devin 云端**：读取云端会话完整记录、把云端会话的规划搬到本地继续执行、发消息/新建云端会话——基于官方 Devin MCP 端点（`mcp.devin.ai`）。

## 🌉 为什么

云端 Devin 会话的规划、决策、进度都存在 `app.devin.ai` 里，本地桌面/CLI agent 默认看不到。这个插件让本地 agent「云感知」：

- 🔍 **读**——拉出任意云端会话的完整对话（「那个搭 SEO 页面体系的会话到底干了什么？」）
- 📦 **接班**——把云端会话的规划搬到本地执行，接着干
- 💬 **通信**——不打开网页就能给云端会话发消息、新建云端会话

## 📁 目录结构

```
.devin-plugin/plugin.json   # 插件清单 + MCP 服务器声明
.mcp.json                   # 根惯例 MCP 配置（其他 MCP 客户端也能直接用）
skills/
  cloud-session-read/       # 解析会话 → 拉记录 → 如实汇报
  cloud-handoff/            # 云端→本地接班流：简报 → 克隆 → 继续执行
```

插件把**官方 Devin MCP server** 接进你的会话（配你自己的 API key），解锁私有工具集：

| 领域 | 工具 |
|---|---|
| 🗂️ 会话 | `devin_session_search` · `devin_session_events` · `devin_session_interact`（发消息/sleep/终止）· `devin_session_create` · `devin_session_gather` · `devin_session_insights` |
| 🧠 平台 | `devin_knowledge_manage` · `devin_playbook_manage` · `devin_schedule_manage` · `devin_automation_manage` |
| 🛡️ 运维 | `devin_review_manage` · `devin_code_scan_manage` · `devin_oncall_manage` · `devin_billing_tag_manage` |

## 🚀 配置

### 1. 拿凭证

- **API key** — <https://app.devin.ai> → **Settings → Devin API** → 创建 personal access token（`cog_` 前缀），或 org 级 service-user key
- **Org ID** — 同一页面，格式 `org-<32位hex>`

### 2. 提供凭证（二选一）

**方案 A — 凭证占位符（推荐）：** 清单里声明了 `Authorization: Bearer ${DEVIN_API_KEY}` 和 `X-Org-Id: ${DEVIN_ORG_ID}`。Devin Cloud 里打开插件的 **Connect** 流程填两个值；本地就导出环境变量：

```bash
export DEVIN_API_KEY='cog_...'
export DEVIN_ORG_ID='org-...'
```

**方案 B — 用户级配置写明文头**（所有 CLI 版本都可用，包括不支持 header env 展开的）。写入 `~/.config/devin/mcp_config.json`（Windows：`%APPDATA%\devin\mcp_config.json`）：

```json
"devin-cloud": {
  "url": "https://mcp.devin.ai/mcp",
  "transport": "http",
  "headers": {
    "Authorization": "Bearer cog_...",
    "X-Org-Id": "org-..."
  }
}
```

> ⚠️ key 永远不要提交进仓库。两种方案都只放在你本地环境里。

### 3. 安装插件

```bash
devin plugins install wookat/devin-cloud-bridge
# 开发调试：
devin plugins install --local /path/to/devin-cloud-bridge
```

装好后新会话自动获得 `mcp__devin-cloud__*` 工具 + `/devin-cloud-bridge:cloud-session-read`、`/devin-cloud-bridge:cloud-handoff` 两个技能（贴会话链接或问云端会话时也会自动触发）。

### 4. 一句话安装（扔给任何 Devin 会话）

不用手动装——把这段 prompt 直接发给本地或云端的 Devin 会话，它自己装自己配：

```text
安装插件 https://github.com/wookat/devin-cloud-bridge —— 执行
`devin plugins install wookat/devin-cloud-bridge`，然后问我要
DEVIN_API_KEY（`cog_` 开头的 token）和 DEVIN_ORG_ID（`org-…`），
以明文 header 写进用户级 mcp_config.json 的 `devin-cloud` 条目里
（不要提交进任何仓库），最后对该 server 调一次 tools/list 验证。
```

`plugins install` 可能需要先 `devin auth login`（浏览器登录一次）——那是 CLI 本体登录，和 API key 是两回事。

## 👥 多账号 / 多 org

一个 `devin-cloud` 条目对应一个 org。有多个 Devin 账号就每账号加一条——URL 相同、凭证不同、命名为 `devin-cloud-<标签>`：

```json
"devin-cloud-acc1": { "url": "https://mcp.devin.ai/mcp", "transport": "http",
  "headers": { "Authorization": "Bearer cog_acc1...", "X-Org-Id": "org-acc1..." } },
"devin-cloud-acc2": { "url": "https://mcp.devin.ai/mcp", "transport": "http",
  "headers": { "Authorization": "Bearer cog_acc2...", "X-Org-Id": "org-acc2..." } }
```

工具按 server 名区分（`mcp__devin-cloud-acc1__*` / `mcp__devin-cloud-acc2__*`），一个本地会话就能同时读写所有账号的云端会话。会话本身不能跨 org 转移，但**规划可以**：在 A 账号读会话记录 → 用接班简报作 prompt 在 B 账号 `devin_session_create`（`cloud-handoff` 技能已覆盖此流程）。

## 🧭 用法

- **「读一下这个云端会话 https://app.devin.ai/sessions/\<id\>」** → agent 拉出完整记录
- **「云端那个会话的规划搬到本地继续做」** → 接班流：简报 → 克隆/checkout → 经你确认后继续执行
- **「把 X 会话放到我另一个账号上继续」** → A 上读 → B 上建
- **「给 528bbbd2 会话发条消息让它收工」** → `devin_session_interact`（写操作会先确认）

## ⚠️ 注意

- 读取免费；**发消息/新建/终止会话消耗 ACU**——技能流程里都会先问
- 会话归属特定 org；报 404 通常是 `DEVIN_ORG_ID` 不对，不是会话不存在
- 云端 VM 上未提交/未 push 的改动本地拿不到——接班技能会在关键处如实标注
- 非官方社区插件，基于[公开 API 文档](https://docs.devin.ai/api-reference)与官方 MCP 端点；与 Cognition AI 无关

## 📄 License

[MIT](./LICENSE)

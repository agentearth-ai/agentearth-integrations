---
name: "agentearth-social-listening"
description: "Find public posts and reactions across social platforms. Search Reddit, X, TikTok, Instagram, Threads, Quora, and Lemon8 where supported, or find Instagram Reels and YouTube Shorts and read comments on a YouTube video. Look up public Facebook posts, posts from a named LinkedIn company or public Telegram channel, public X Spaces, or available TikTok LIVE rankings. For mainland China, use the search, link, or account options for Douyin, RedNote (Xiaohongshu), Weibo, Bilibili, Kuaishou, Zhihu, Baidu Tieba, WeChat Official Accounts and Channels, and WeChat In-App Search. Check public Douyin LIVE rankings or keyword trends in Douyin Index and WeChat Index, where available. Find public posts or videos on Pipixia, Toutiao, Xigua Video, and Youku, or see viewer reactions on Douban."
---

# agentearth-social-listening

Find public posts and reactions across social platforms. Search Reddit, X, TikTok, Instagram, Threads, Quora, and Lemon8 where supported, or find Instagram Reels and YouTube Shorts and read comments on a YouTube video. Look up public Facebook posts, posts from a named LinkedIn company or public Telegram channel, public X Spaces, or available TikTok LIVE rankings. For mainland China, use the search, link, or account options for Douyin, RedNote (Xiaohongshu), Weibo, Bilibili, Kuaishou, Zhihu, Baidu Tieba, WeChat Official Accounts and Channels, and WeChat In-App Search. Check public Douyin LIVE rankings or keyword trends in Douyin Index and WeChat Index, where available. Find public posts or videos on Pipixia, Toutiao, Xigua Video, and Youku, or see viewer reactions on Douban.

Call the tools listed below from the connected `agentearth-plugin` MCP server.
Use the tool names exposed by Hermes for that server.
Hermes handles authentication. Do not send HTTP requests yourself.

## Available Tools

### GetAccountOverview

Returns the authenticated user's account info. Takes no arguments.

**Response fields**: `user_id`, `user_name`, `key_name`, `credit` (remaining balance), `error_no` (`0` = success).

Call this directly — without `RecommendTools` first — whenever the user asks
about their AgentEarth account, user ID, API key name, or credit balance.

### GetUsageRecords

Query the authenticated account's spending, tool calls and credit transactions. Call the tool directly for billing questions. It defaults to a seven-day usage summary; data can lag by one minute.

Use `type="usage"` (the default) to review tool-call spending and call history. Use `type="funds"` to review account credit transactions, such as recharges, rewards, and deductions. Use `type="all"` to review both and explain a balance change. Use `view="summary"` (the default) for totals, or `view="records"` for paginated records. The default time range is seven days.

Pending calls have zero charged credit and a separate `reserved_credit`. `status` filters call details; totals cover the whole query window.

### RecommendTools - discover tools for a task

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| query | string | Yes | Natural-language description of the task |
| limit | integer | - | Maximum number of tools to return (1–50), default 5 |

Returns a list of candidate tools, each with:

- `tool_name` — the tool's identifier
- `tool_url` — opaque URL to pass to `ExecuteTool`; **never modify it**
- `description`, `when_to_use` — what the tool does and when to pick it
- `credit` — cost of one call, use this plus task fit to choose among candidates
- `input_schema` — JSON Schema for `ExecuteTool`'s `params`
- `associated_tools` — optional companion tools (e.g. a geocoding/ID lookup tool) that may need to be called first if a required field can't be filled directly from user input

Call this before `ExecuteTool` for any external-tool task. Read every
candidate's `input_schema` and `associated_tools` before picking one — some
fields require an ID or code obtained from a companion lookup tool rather
than free text (e.g. a weather tool that needs a location ID resolved by a
geocoding tool first).

### ListTools - browse tools without semantic search

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| page | integer | - | Page number starting at 1, default 1 |
| page_size | integer | - | Items per page (1–100), default 20 |
| keyword | string | - | Filter by keyword in the name or description |
| sort | string | - | Sort order |

Use when the user wants to browse or search tools by name/category rather
than describe a task.

### GetToolDetail - inspect one tool by exact name

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| tool_name | string | Yes | Exact tool name |

Returns the same shape as one entry from `RecommendTools`/`ListTools`
(`tool_url`, `description`, `input_schema`, etc). Use when you already know
the exact `tool_name` (e.g. from `associated_tools`, or the user named a
specific tool) and don't need semantic search.

### ExecuteTool - run a tool

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| tool_url | string | Yes | Exact URL returned by `RecommendTools` or `GetToolDetail`. Do not edit it, append anything to it, or reorder its parameters. |
| params | object | - | Arguments built from the selected tool's `input_schema` |

Must be called after `RecommendTools` or `GetToolDetail`. `tool_url` must be
passed through byte-for-byte as returned — do not edit, re-encode, or
reconstruct it. Build `params` strictly from the selected tool's
`input_schema.properties`; validate `required`, `type`, `enum`, and
`additionalProperties` before calling.

## Examples

- “How many credits did I spend over the past seven days?” → `GetUsageRecords({"type":"usage","view":"summary"})`. Read `summary.total_credit`.
- “Show me each tool call from the past seven days and how many credits it cost.” → `GetUsageRecords({"type":"usage","view":"records"})`. Read the paginated tool call records.
- “Why did my balance change? Show me every transaction.” → `GetUsageRecords({"type":"all","view":"records"})`. Explain the change using `balance_change`, tool call records, and credit transaction records.
- "What's my AgentEarth credit balance?" → call `GetAccountOverview()` directly (no `RecommendTools`).
- "Use AgentEarth to check today's weather in Shanghai" → `RecommendTools({"query": "current weather in Shanghai"})` → pick a weather tool from the candidates → if its `input_schema` needs a location ID/adcode rather than a free-text city name, call the companion lookup tool named in `associated_tools` first → `ExecuteTool({"tool_url": <candidate's tool_url>, "params": {...}})`.
- "Use AgentEarth to check the current Bitcoin price" → `RecommendTools({"query": "current bitcoin price"})` → `ExecuteTool` with the chosen finance tool's `tool_url` and schema-derived `params`.
- "List the AgentEarth tools related to maps" → `ListTools({"keyword": "map"})` to browse by name instead of describing a task.
- User names a specific tool exactly (e.g. "run ae_qweather_geo_top_city") → `GetToolDetail({"tool_name": "ae_qweather_geo_top_city"})` instead of `RecommendTools`.

## Workflow

1. Account questions (user ID, user name, key name, credit balance) → call
   `GetAccountOverview` directly. For spending, billing history or changes in balance, call `GetUsageRecords` directly.
2. External-tool tasks → call `RecommendTools` with a natural-language
   description of the task, or `ListTools`/`GetToolDetail` when the user
   wants to browse or inspect tools by name.
3. Review the candidates' `tool_url`, `description`, `when_to_use`, and
   `input_schema`. Pick the best fit by task relevance, schema clarity, and
   `credit` cost; keep a fallback candidate when one exists.
4. If a required param needs an ID/code from a companion tool (see
   `associated_tools`), call that tool first.
5. Call `ExecuteTool` with the exact `tool_url` and `params` built from
   `input_schema`.
6. Never invent a required value (URL, ID, token, code snippet, or other
   concrete input) that the user hasn't provided — ask the user instead.

## Reading the Result

`ExecuteTool`'s MCP result has an `isError` field — that is the reliable
success/failure signal, not a fixed field inside the payload:

- **`isError` absent/false** — the call succeeded. The content is the
  **raw response of the underlying third-party API**, in that API's own
  format (it may or may not contain something resembling `error_no`/status
  fields of its own — don't assume a single universal shape across tools).
- **`isError: true`** — the call failed. The error detail comes back
  wrapped as `<tool_output>...</tool_output>`, often followed by a
  `<platform_annotation>` with a suggestion (e.g. "revise your request
  parameters based on the upstream provider's error message"). Use that
  detail to correct `params` and retry, or fall back to another candidate
  from `RecommendTools` if the issue isn't fixable from user-provided input.

`GetUsageRecords`/`GetAccountOverview` and `GetToolDetail`/`RecommendTools`/`ListTools` (the
AgentEarth-native lookups, as opposed to `ExecuteTool` passthrough results)
do use `error_no == 0` as their own success signal.

## Credentials

If the AgentEarth MCP tools are unavailable, ask the user to connect in a
terminal for the same Hermes profile:

```bash
hermes mcp add agentearth-plugin --url https://agentearth.ai/mcp-server/r/FbtaiJMv --auth oauth
```

Hermes opens the browser for AgentEarth sign-in and manages the tokens. Use
`hermes mcp login agentearth-plugin` to sign in again after setup. Start a new chat
if needed so Hermes can discover the connected server's tools. Never ask the
user to paste credentials in chat.

## Incorrect Flow

- Calling `ExecuteTool` before `RecommendTools`/`GetToolDetail`.
- Modifying, re-encoding, or reconstructing `tool_url`.
- Inventing required params instead of asking the user.
- Treating `error_no == 0` as the success signal for `ExecuteTool` results —
  use `isError` instead.
- Making raw HTTP requests to AgentEarth endpoints yourself, or asking the
  user for an API key — use the six MCP tools above through the profile's AgentEarth OAuth connection.

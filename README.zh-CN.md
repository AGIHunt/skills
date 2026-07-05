# AGI HUNT Skills

[English](README.md) | **中文**

[AGI HUNT](https://agihunt.info) 官方 agent skill 集合。本仓库会持续收录更多 skill——目前刚起步,`agihunt`(实时 AI 资讯)是第一个。

## Skill 列表

| Skill | 用途 | 安装 |
|---|---|---|
| [`agihunt`](agihunt/SKILL.md) | 让你的 AI Agent 直接获取实时 AI 资讯、每日 AI 日报、各主题频道最热/最新帖子 | 见下 |

## agihunt — 给 Agent 的实时 AI 资讯

让你的 AI Agent(Claude Code、Codex 等任意能联网执行命令的 Agent)直接接入 AGI HUNT 的实时 AI 信号与每日 AI 日报。

**接入只需一句话。** 发给你的 Agent:

> 请帮我接入 AGI HUNT 实时 AI 资讯:阅读 https://agihunt.info/agent/v1/skill.md 并按其中的说明完成安装与授权。

Agent 会自己安装 skill,给你一个授权链接(用 Google 登录,点一次「授权」),然后自动领取并保存专属 API 密钥。之后直接问它「AI 日报」「AGI Hunt 最热的 AI 投融资动态」即可。

- 接入页 / 密钥管理:https://agihunt.info/access
- 权威分发源(服务器强制版本升级):https://agihunt.info/agent/v1/skill.md
- 本仓库为镜像,便于浏览与审阅;以服务器版本为准。

## 使用守则

数据端点仅覆盖近 3 天;限速 0.5 req/s(突发 10)、每密钥每天 1000 次;skill 内置 10 分钟本地缓存。转载请注明出处 **AGI HUNT · agihunt.info**。

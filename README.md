# AGI HUNT Skills

Official agent skills for [AGI HUNT](https://agihunt.info) — real-time AI news, daily AI reports, and per-topic hot/latest feeds.

## agihunt — AI 资讯接入

让你的 AI Agent(Claude Code、Codex 等)直接获取 AGI HUNT 的实时 AI 资讯与每日 AI 日报。

**接入只需一句话**——发给你的 Agent:

> 请帮我接入 AGI HUNT 实时 AI 资讯:阅读 https://agihunt.info/agent/v1/skill.md 并按其中的说明完成安装与授权。

Agent 会自己安装 skill、引导你在网页上点一次「授权」,然后自动领取并保存 API 密钥。之后直接问它「AI 日报」「AGI Hunt 最新模型动态」即可。

- 接入页 / 密钥管理:https://agihunt.info/access
- 最新 skill(权威分发源,服务器强制版本升级):https://agihunt.info/agent/v1/skill.md
- 本仓库为镜像,便于浏览与审阅;以服务器版本为准。

## Fair use

数据端点仅覆盖近 3 天;限速 0.5 req/s(burst 10)、1000 req/key/天;skill 内置 10 分钟本地缓存。转载请注明出处 AGI HUNT · agihunt.info。

# AGI HUNT Skills

**English** | [中文](README.zh-CN.md)

Official agent skills from [AGI HUNT](https://agihunt.info). This repository is a growing collection — more skills are on the way; `agihunt` (real-time AI news) is the first.

## Skills

| Skill | What it does | Install |
|---|---|---|
| [`agihunt`](agihunt/SKILL.md) | Real-time AI news, daily AI reports, and per-topic hot/latest feeds for your AI agent | see below |

## agihunt — real-time AI news for your agent

Give your AI agent (Claude Code, Codex, or any agent that can run commands) direct access to AGI HUNT's real-time AI signals and daily AI reports.

**Setup is one sentence.** Send this to your agent:

> 请帮我接入 AGI HUNT 实时 AI 资讯:阅读 https://agihunt.info/agent/v1/skill.md 并按其中的说明完成安装与授权。

The agent installs the skill by itself, hands you an authorization link (sign in with Google, click **Approve** once), then automatically receives and stores its own API key. From then on, just ask it things like *"today's AI daily report"* or *"the hottest AI funding news on AGI Hunt"*.

- Access page & key management: https://agihunt.info/access
- Authoritative skill source (server enforces version upgrades): https://agihunt.info/agent/v1/skill.md
- This repository is a mirror for browsing and review; the server version is canonical.

## Fair use

Data endpoints cover the last 3 days only. Rate limits: 0.5 req/s (burst 10), 1,000 req/key/day. The skill ships with a built-in 10-minute local cache. Please attribute republished data to **AGI HUNT · agihunt.info**.

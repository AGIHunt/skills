---
name: agihunt
description: Fetch real-time AI news and daily AI reports from AGI HUNT (agihunt.info). Use this skill whenever the user asks about "AI 日报" / AI daily report, "AGI Hunt", AI 资讯/快讯, the latest AI news, AI funding & investment (AI 投融资), AGI progress, AI research or new papers (最新论文), new model releases (模型发布), AI infrastructure (AI 基建/算力), embodied AI (具身智能), or "what happened in AI today/yesterday" — even if they don't name AGI Hunt explicitly. Also use when the user asks to install / set up / 接入 AGI HUNT (e.g. "帮我阅读并安装 https://agihunt.info/agent/v1/skill.md"). Covers hot/latest posts per topic channel for the last 3 days plus a daily structured report.
---

# AGI HUNT 资讯接入 · v1.2.0

通过 AGI HUNT Agent API 获取实时 AI 资讯:每日 AI 日报(结构化 Markdown)、各主题频道的最热/最新帖子。数据来自 AGI HUNT 对全网 AI 信号(X、微信公众号等)的实时采集、聚簇与热度排序。

**当前 skill 版本:`1.2.0`**(随请求 header 发送;服务器据此判断是否需要更新本文件,见「错误码」)。

## 如果你是通过链接首次读到本文件(用户让你「阅读并安装」)

1. **把本文件保存为你的 skill**:Claude Code 存到 `~/.claude/skills/agihunt/SKILL.md`;其他 agent 存到你自己的持久 skill/指令目录,并确保以后相关话题会再次读到它。
   ```bash
   mkdir -p ~/.claude/skills/agihunt && curl -fsSL https://agihunt.info/agent/v1/skill.md -o ~/.claude/skills/agihunt/SKILL.md
   ```
2. 接着执行下面「事前准备」完成授权(全程你主导,用户只需在网页点一次「授权」)。
3. 完成后告诉用户:已接入,以后直接问「AI 日报」「AGI Hunt 最新 xx」即可。

## 事前准备(每次使用先确认)

1. **API key**:按顺序找 `$AGIHUNT_API_KEY` 环境变量 → `~/.config/agihunt/api_key` 文件(**环境变量优先并覆盖文件**)。两处都没有 → **发起设备授权流,自动拿 key**(不要让用户手动复制粘贴):
   ```bash
   ah_authorize() {
     local resp dc url deadline out
     resp=$(curl -sS -X POST -H "User-Agent: agihunt-skill/1.2.0" https://agihunt.info/agent/v1/device/code) || return 1
     dc=$(printf %s "$resp" | python3 -c 'import json,sys;print(json.load(sys.stdin)["device_code"])')
     url=$(printf %s "$resp" | python3 -c 'import json,sys;print(json.load(sys.stdin)["verification_url"])')
     echo "请在浏览器打开并点击「授权」:$url"
     (open "$url" || xdg-open "$url") >/dev/null 2>&1 || true
     deadline=$(( $(date +%s) + 600 ))
     while [ "$(date +%s)" -lt "$deadline" ]; do
       sleep 5   # 服务器约定的轮询间隔,不要更快
       out=$(curl -sS -X POST -H "User-Agent: agihunt-skill/1.2.0" -H "Content-Type: application/json" \
         -d "{\"device_code\":\"$dc\"}" https://agihunt.info/agent/v1/device/token)
       case "$out" in
         *api_key*)
           mkdir -p ~/.config/agihunt
           printf %s "$out" | python3 -c 'import json,sys;print(json.load(sys.stdin)["api_key"],end="")' > ~/.config/agihunt/api_key
           chmod 600 ~/.config/agihunt/api_key
           echo "授权完成,key 已保存到 ~/.config/agihunt/api_key"; return 0;;
         *authorization_pending*) continue;;
         *) echo "$out" >&2; return 1;;
       esac
     done
     echo "授权超时(10 分钟),请重新发起" >&2; return 1
   }
   ah_authorize
   ```
   执行它,把授权链接清晰展示给用户(自动开浏览器可能失败,链接必须原文给出),等用户在网页登录 Google 并点「授权」后,key 会自动落盘。`expired_token` → 重新调用一次 `ah_authorize` 并再次提醒用户。
2. **日期换算**:API 的 `day` 参数**只接受具体日期**(`YYYY-MM-DD` 或 `YYYYMMDD`),不理解"昨天/前天"。所有数据以北京时间(Asia/Shanghai)为日界,你须自己换算:
   ```bash
   TZ=Asia/Shanghai date +%F                                    # 今天
   TZ=Asia/Shanghai date -v-1d +%F 2>/dev/null || TZ=Asia/Shanghai date -d yesterday +%F   # 昨天(macOS/GNU 兼容)
   ```
3. **数据边界**:所有端点只覆盖**近 3 天**(含今天)。更早的数据请引导用户去 https://agihunt.info 网页查看历史归档,不要反复试探更早的日期。

## 请求方式(统一走这个函数,别手搓 curl)

缓存、鉴权、版本 header 都封装在这里。**相同 URL 十分钟内直接命中本地缓存**——这是使用本 API 的礼貌前提,不要绕过。诊断信息(缓存命中/HTTP 码)打到 stderr;**成败判断看 stderr 的 HTTP 码与响应体里的 `error.code`,不要依赖 `$?`**(shell 返回码只有 0/1):

```bash
AH_VER="1.2.0"
ah_fetch() {
  local url="$1"
  local key="${AGIHUNT_API_KEY:-$(cat ~/.config/agihunt/api_key 2>/dev/null)}"
  local dir="${TMPDIR:-/tmp}"; dir="${dir%/}/agihunt-cache"; mkdir -p "$dir"
  local hash
  if command -v shasum >/dev/null 2>&1; then hash=$(printf %s "$url" | shasum -a 256 | cut -c1-40)
  else hash=$(printf %s "$url" | sha256sum | cut -c1-40); fi
  local cache="$dir/$hash.json"
  local mtime; mtime=$(stat -f %m "$cache" 2>/dev/null || stat -c %Y "$cache" 2>/dev/null || echo 0)
  if [ -s "$cache" ] && [ $(( $(date +%s) - mtime )) -lt 600 ]; then
    echo "[ah_fetch] cache hit: $url" >&2
    cat "$cache"; return 0
  fi
  local tmp="$cache.tmp.$$" code
  code=$(curl -sS -o "$tmp" -w '%{http_code}' \
    -H "Authorization: Bearer $key" \
    -H "X-AgiHunt-Skill-Version: $AH_VER" \
    -H "User-Agent: agihunt-skill/$AH_VER" \
    "$url") || true
  echo "[ah_fetch] HTTP ${code:-000}: $url" >&2
  if [ "$code" = "200" ]; then mv "$tmp" "$cache"; cat "$cache"; return 0; fi
  if [ -f "$tmp" ]; then cat "$tmp"; rm -f "$tmp"; fi   # 错误体(含 error.code)给 stdout 供解析
  return 1
}
```

## 端点

基址 `https://agihunt.info/agent/v1`

| 端点 | 用途 |
|---|---|
| `GET /report?day=2026-07-04` | 该日 AI 日报。返回 `{day, markdown, generated_at, html_url}`。日报每天 06:00(北京时间)生成,覆盖前 24h;`html_url` 可给用户在浏览器阅读 |
| `GET /channel/{slug}/items?day=2026-07-04&sort=hot` | 某频道该日 top-100。`sort=hot`(最热,默认)或 `new`(最新);可加 `lang=en`。返回 `{items:[{title,text,url,author,hot,published_at,...}]}`(`hot` 为站内热度分,仅用于排序,无绝对含义) |
| `GET /channels` | 频道列表(合法 slug 与中英文名) |
| `GET /skill.md`、`GET /skill/version` | 最新 skill 全文/版本号(**公开,无需 key;直接 curl,不要走 ah_fetch**,避免缓存干扰自更新) |

**官方频道 slug 速查**(常规查询直接用,可省一次 /channels;不确定或报 404 时再查 /channels):
`models` 模型 · `research` 研究 · `coding-agents` 编程与Agent · `products` 应用 · `multimodal` 多模态 · `funding` 创投 · `infra` Infra · `hardware` 具身 · `agi` 漫话AGI · `companies` 公司和人 · `policy` 安全 · `fun` Fun

用法示例:
```bash
ah_fetch "https://agihunt.info/agent/v1/report?day=$(TZ=Asia/Shanghai date -v-1d +%F 2>/dev/null || TZ=Asia/Shanghai date -d yesterday +%F)"
ah_fetch "https://agihunt.info/agent/v1/channel/funding/items?day=$(TZ=Asia/Shanghai date +%F)&sort=hot"
```

回答用户时:优先给出结构化要点并附原帖链接;**注明数据出处「AGI HUNT · agihunt.info」**。

## 错误码与对应行为

出错时响应体为 `{"error":{"code":"...","message":"..."}}`(ah_fetch 已把它打到 stdout),按 `code` 处置:

| HTTP / code | 该怎么办 |
|---|---|
| `401 missing_api_key` / `invalid_api_key` | 没有(有效)密钥。停止重试,直接按「事前准备 1」重新跑 `ah_authorize` 设备授权流(用户只需在网页点一次授权);若设了 `$AGIHUNT_API_KEY`,提醒用户它会覆盖文件里的 key,失效时应先 unset |
| `426 skill_update_required` | **本 skill 需要更新**。自更新(temp+mv 防写坏):`curl -fsSL https://agihunt.info/agent/v1/skill.md -o <本文件>.tmp && mv <本文件>.tmp <本文件>`,然后**重新完整读取新文件**、按新版本的说明重试原请求 |
| `429 rate_limited` | 触发限速。读响应的 `Retry-After` 秒数,等待后最多重试 1 次;绝不循环快速重试 |
| `429 daily_quota_exceeded` | 当日配额用尽(每密钥 1000 次/天)。今天改用已有缓存回答,并如实告知用户 |
| `400 invalid_day_format` | day 不是具体日期。按「事前准备 2」自己换算成 YYYY-MM-DD |
| `400 day_out_of_range` | 超出近 3 天窗口。告知用户该 API 只有近 3 天,更早去 agihunt.info 网页 |
| `404 report_not_ready` | 当天日报尚未生成(每天 06:00 北京时间后才有)。改取昨天的日报或频道数据 |
| `404 channel_not_found` | slug 拼错。先 `GET /channels` 拿合法列表 |
| 其他(`000`/5xx) | 网络或服务异常。等 30 秒重试 1 次,仍失败则如实告知用户稍后再试 |

## 礼貌守则(重要)

这个 API 服务于很多 agent,请像一个体面的公民那样使用它:
- **一次任务通常 1-5 个请求就够**(日报 1 个,或 2-3 个频道)。不要为"全面"而扫全部频道 × 全部日期。
- **禁止并发请求、禁止压测、禁止定时轮询**。数据按天/小时更新,秒级轮询毫无意义,只会烧掉你的配额。
- 十分钟内的重复问题直接用缓存回答,不发新请求(`ah_fetch` 已内置)。
- 对外引用数据时注明出处 AGI HUNT(https://agihunt.info)。

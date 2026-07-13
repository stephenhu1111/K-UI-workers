## 🌟 项目赞助商 (Sponsor)

<p align="center">
  <a href="https://derouter.ai?ref=0oZZ1HVc" target="_blank">
    <b>DeRouter</b> — 基于区块链的纯正透明大模型 API 网关
  </a>
</p>

**DeRouter** 通过区块链技术保证 Claude、GPT 官方 API 的**纯正与透明**，杜绝掺水降智问题。目前 Claude、GPT 在能力上仍领先国内大模型。

- 🔗 官网：https://derouter.ai?ref=0oZZ1HVc
- 🐦 Twitter：https://x.com/derouter_net
- 💰 **有多余的 Claude 账号**：可托管到平台赚取收益
- ⚡ **有 API 需求**：可使用其平台，价格为官方 API 的 **1-2 折**

<br>

<p align="center">
  <a href="https://bytevirt.com/aff.php?aff=209" target="_blank">
    <b>ByteVirt</b> — 始于方寸字节，成就无限云端
  </a>
</p>

**ByteVirt** 是一家专注于高性价比云服务器的 VPS 厂商，提供稳定可靠的虚拟化云端主机，适合部署 KUI 节点、探针 bash 及各类自建服务。

- 🔗 官网：https://bytevirt.com/aff.php?aff=209
- 🖥️ **多地域机房**：可按需选择节点位置，满足代理与监控部署需求
- ⚡ **稳定高速**：优质网络与虚拟化性能，保障服务长期在线

---

# KUI x Server Monitor Pro

KUI 是一套由 Cloudflare Pages、Pages Functions、D1、独立 Realtime Worker、Durable Objects 和 VPS Agent 组成的代理聚合、服务器监控与住宅双隧道控制面板。

## 当前架构

KUI 至少需要部署 Cloudflare Pages 和一个生产 D1。Realtime Worker 是推荐的第二个 Cloudflare 应用；启用后必须与 Pages 共享同一个生产 D1：

| 组件 | 部署位置 | 必需 | 作用 |
|---|---|---:|---|
| KUI 主站 | Cloudflare Pages | 是 | 前端、Pages Functions、登录、配置、订阅、D1 持久化、Agent 安装与更新 |
| Realtime Worker | Cloudflare Workers | 推荐 | Agent 与浏览器 WebSocket、Durable Objects、1-5 秒实时状态和配置通知 |
| D1 | Cloudflare D1 | 是 | 用户、VPS、节点、配置、流量、探针和住宅状态的长期数据 |
| Core Agent | VPS | 是 | 系统监控、sing-box 配置、流量统计和 Core WebSocket |
| Proxy Manager | VPS | Full Deploy 默认安装 | OpenVPN 主备隧道、SOCKS5/HTTP 代理和 Proxy WebSocket |

```text
                                   同一个生产 D1
                               ┌──────────────────┐
                               │                  │
浏览器 ── HTTPS ── Pages ──────┤                  ├──── Realtime Worker
  │              Functions     │                  │       │
  │                            └──────────────────┘       │
  └──────── Dashboard WebSocket ──────────────────────────┤
                                                         │
VPS Core Agent ───────────── Core WebSocket ───── VpsPresence DO
VPS Proxy Manager ────────── Proxy WebSocket ───── VpsPresence DO
                                                         │
                                                   DashboardHub DO
```

Pages + D1 可以单独运行 HTTP 模式；要获得 1-5 秒状态显示、实时配置通知和更低的 Pages 请求量，必须额外部署 Realtime Worker 和两个 Durable Object 类。

## 功能概览

- Vue 3 单页管理后台与公开探针大盘。
- XTLS-Reality、Hysteria2、TUIC、Trojan、H2-Reality、gRPC-Reality、AnyTLS、Naive 和 VLESS-Argo 节点管理。
- 多用户、配额、到期、订阅、第三方订阅和流量统计。
- CPU、内存、磁盘、负载、网速、TCP/UDP 和多线路延迟监控。
- 每台 VPS 独立 Agent Token，不使用管理员密码作为 Agent 凭据。
- Core 与住宅代理每 5 秒通过 WebSocket 上报，通常 1-5 秒显示。
- 配置保存后实时通知 VPS，配置结果返回面板。
- Dashboard WebSocket 健康时停止周期 Pages API 轮询。
- WebSocket 连续断开满 30 秒后自动切换 HTTP fallback，恢复后自动切回。
- VPS Python 组件每小时通过鉴权端点检查更新，执行 SHA256、大小和语法校验后原子替换。

## 支持范围

### Cloudflare

- Cloudflare Pages、Pages Functions 和 D1。
- Cloudflare Workers 与 SQLite-backed Durable Objects，用于实时模式。
- Production 和 Preview 环境相互独立，必须在 Production 环境配置绑定和变量。

### VPS

- Debian、Ubuntu、Alpine Linux。
- `x86_64` 或 `aarch64`。
- 必须使用 root 执行 Full Deploy。
- 必须允许出站 HTTPS。
- 住宅代理需要 `/dev/net/tun` 和服务商允许 TUN/TAP。
- Full Deploy 不支持其他发行版；仓库中的部分脚本存在其他包管理器代码，不代表正式支持。

## 安全准备

部署前准备以下随机强凭据：

| 名称 | 建议 |
|---|---|
| `ADMIN_USERNAME` | 不要使用 `admin` |
| `ADMIN_PASSWORD` | 随机 24 位以上 |
| `PROXY_USER` | 非常用用户名 |
| `PROXY_PASS` | 随机 32 位以上 |
| `CRON_SECRET` | 启用定时离线检查时使用，随机 32 位以上 |
| `TG_WEBHOOK_SECRET` | 启用 Telegram Webhook 时使用，随机 32 位以上 |

可以在本地生成随机值：

```bash
openssl rand -base64 32
```

重要说明：

- Pages 代码在缺少管理员变量时存在兼容默认值 `admin/admin`，生产环境必须显式设置 `ADMIN_USERNAME` 和 `ADMIN_PASSWORD`。
- `ADMIN_PASSWORD`、`PROXY_USER`、`PROXY_PASS`、`CRON_SECRET`、`TG_BOT_TOKEN`、`TG_CHAT_ID`、`TG_WEBHOOK_SECRET` 应使用 Cloudflare Secret。
- Realtime Worker 的管理员用户名和密码必须与 Pages 完全相同。
- Full Deploy Command 包含服务器专属 Agent Token，属于敏感信息，不要发到 Issue、聊天群或公开日志。
- 住宅代理凭据可能由鉴权 API 返回给管理员或对应 Agent；泄露后应立即轮换 `PROXY_PASS`。

# 完整部署指南

推荐严格按照以下顺序部署：

1. Fork 仓库并确定 Pages 生产分支。
2. 创建一个生产 D1。
3. 部署 Pages，但先不要设置 `REALTIME_URL`。
4. 给 Pages 绑定 D1、配置变量和 Secret，然后重新部署。
5. 登录 Pages，触发 D1 schema 初始化并验证 HTTP 模式。
6. 部署独立 Realtime Worker，绑定同一个 D1 和两个 Durable Object。
7. 验证 Worker `/health`、凭据和来源配置。
8. 在 Pages 设置 `REALTIME_URL`，重新部署 Pages，正式启用实时模式。
9. 在面板添加 VPS，执行面板生成的 Full Deploy Command。
10. 验证 Pages、Worker、DO、Core Agent、Proxy Manager 和 fallback。

## 第一步：Fork 仓库并选择生产分支

1. Fork 本仓库到自己的 GitHub。
2. 确认要用于生产的分支包含最新代码。
3. 在 Cloudflare Pages 创建项目时，将该分支明确设置为 Production branch。
4. 后续只有推送到 Production branch 并完成 Pages 生产发布，VPS 才能通过 `/api/agent_update` 获取新组件。

不要把 Preview 环境当作 Production。Preview 的变量、Secret 和 D1 绑定与 Production 独立。

## 第二步：创建生产 D1

1. 登录 Cloudflare Dashboard。
2. 进入 **Storage & databases** → **D1 SQL database**。
3. 创建数据库，例如 `kui-db`。
4. 记录数据库名称和 database ID，部署 Realtime Worker 时还会使用。

KUI 没有独立 SQL migration 文件。Pages Functions 仅在会调用 `ensureDbSchema()` 的路由中执行幂等建表和兼容字段升级，例如 `/api/login`、`/api/probe/*`、`/api/report` 和 `/api/config`。

正常不需要手工建表。Pages 发布完成后，请执行一次管理员登录，确保 schema 初始化。仅访问静态首页不一定触发初始化。

主要表包括：

```text
servers
users
nodes
traffic_stats
sys_config
proxy_ctrl_servers
server_logs
proxy_slot_map
report_receipts
probe_settings
probe_servers
proxy_servers
third_party_subscriptions
third_party_nodes
```

运行时 DDL 是兼容机制，不是完整的事务化版本迁移系统。升级重要生产实例前建议备份 D1。

## 第三步：部署 Cloudflare Pages

1. 进入 **Workers & Pages** → **Create application** → **Pages** → **Connect to Git**。
2. 选择 Fork 后的仓库。
3. 设置 Production branch。
4. 构建设置：

| 设置 | 值 |
|---|---|
| Framework preset | `None` |
| Build command | `exit 0` |
| Build output directory | `.` |
| Root directory | 留空；仅当仓库位于 monorepo 子目录时填写该子目录 |

5. 点击部署。

仓库根目录必须包含：

```text
index.html
functions/api/[[path]].js
vps/
realtime/
```

主应用必须部署为 Cloudflare Pages。`/api/agent_update` 使用 Pages 自动提供的 `ASSETS` 绑定读取当前发布的 VPS 组件；如果改成普通 Worker 且没有等价的 Assets 绑定，该接口会返回 `503`。

## 第四步：配置 Pages Production 环境

进入 Pages 项目 **Settings**，确保以下配置作用于 Production。

### D1 绑定

| 类型 | 绑定名 | 选择内容 |
|---|---|---|
| D1 database | `DB` | 第二步创建的生产 D1 |

绑定名必须严格为 `DB`。`ASSETS` 由 Pages 自动提供，不要手工创建。

### Pages 必填变量和 Secret

| 名称 | 类型 | 说明 |
|---|---|---|
| `ADMIN_USERNAME` | Variable | 管理员用户名 |
| `ADMIN_PASSWORD` | Secret | 管理员强密码 |
| `PROXY_USER` | Secret | Full Deploy 住宅代理用户名 |
| `PROXY_PASS` | Secret | Full Deploy 住宅代理密码 |

如果不使用住宅代理，基础监控和节点功能可以不配置 `PROXY_USER`、`PROXY_PASS`；Full Deploy 的住宅阶段和相关 API 将无法正常工作。

### Pages 可选变量

| 名称 | 类型 | 说明 |
|---|---|---|
| `REALTIME_URL` | Variable | Realtime Worker HTTPS 来源；先留空，Worker 验证完成后再填写 |
| `CRON_SECRET` | Secret | `/api/cron_check` 的 Bearer Secret |
| `TG_BOT_TOKEN` | Secret | 服务端离线告警的 Telegram Token fallback |
| `TG_CHAT_ID` | Variable/Secret | 服务端离线告警的 Telegram Chat ID fallback |
| `TG_WEBHOOK_SECRET` | Secret | Telegram Webhook 请求校验 |
| `LEGACY_AGENT_AUTH` | Variable | 仅旧 Agent 迁移时临时设置为精确字符串 `true` |

修改绑定或变量后必须重新部署 Pages。

### 标准部署不要配置的变量

标准部署使用内置 D1 住宅控制器，不要设置：

```text
PROXY_CTRL_URL
PROXY_CTRL_USER
PROXY_CTRL_PASS
PROXY_CTRL_TOKEN
```

代码仍支持外部控制器，但设置 `PROXY_CTRL_URL` 后，Pages 会桥接代理 API，Full Deploy 会检测到 external 模式并停止内置住宅安装。外部模式需要自行部署对应控制器和住宅组件。

`PROXY_PORT` 也不是 Pages 的标准环境变量。内置控制器默认端口是 `7920`，应在住宅代理页面或鉴权 API 中修改。

## 第五步：初始化和验证 Pages

1. 重新部署 Pages。
2. 打开 Pages 域名。
3. 使用 `ADMIN_USERNAME` 和 `ADMIN_PASSWORD` 登录。
4. 登录请求会触发 D1 schema 初始化。
5. 检查以下项目：

| 检查 | 预期 |
|---|---|
| `GET /` | 返回 KUI 页面 |
| `GET /api/probe/public` | 返回 JSON，不出现缺少 `DB` |
| 管理员登录 | 成功进入后台 |
| 已登录后台发起的 `GET /api/data` | 返回 `servers`、`nodes`、`users`；无有效管理员鉴权会返回 `401` |
| 添加 VPS | 生成服务器记录和独立 `agent_token` |

Pages 没有单独 `/health` 接口。请使用实际页面、登录和 API 验证。

此时不要急于设置 `REALTIME_URL`。先确保纯 HTTP 模式可登录、D1 正常、VPS 可以被添加。

## 第六步：部署 Realtime Worker

Realtime Worker 源码是：

```text
realtime/src/index.js
```

不要复制 `functions/api/[[path]].js`，它是 Pages Functions 后端，不是 Realtime Worker。

### 一键部署 Worker + Durable Objects

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/a6216abcd/K-UI/tree/dev/realtime)

点击按钮后，Cloudflare 会克隆 `realtime/` 独立模板并部署：

- `kui-realtime` Worker。
- `VpsPresence` SQLite-backed Durable Object。
- `DashboardHub` SQLite-backed Durable Object。
- `VPS_PRESENCE`、`DASHBOARD_HUB` 绑定。
- `DB` D1 绑定。
- `v1` Durable Object migration。

> Deploy to Cloudflare 按钮只能部署 Worker 应用，不能同时部署 Pages。KUI Pages 仍需按前文单独连接 GitHub 部署。

一键部署页面需要填写：

| 项目 | 要求 |
|---|---|
| `ADMIN_USERNAME` | 与 Pages Production 完全相同 |
| `ADMIN_PASSWORD` | 与 Pages Production 完全相同，作为 Secret |
| `PAGES_ORIGIN` | Pages 完整来源，例如 `https://your-kui.pages.dev`，末尾无 `/` |
| `DB` | 必须最终与 Pages Production 使用同一个 D1 |

如果是一套全新 KUI，可以让一键部署流程创建 `kui-db`，然后在 Pages Production 中把这个数据库绑定为 `DB`。如果 Pages 已经有数据，一键部署后必须进入 Worker **Settings** → **Bindings**，确认 Worker 的 `DB` 指向 Pages 当前使用的同一个 D1 database ID；不要让 Worker 保留一个新的空 D1，否则 Agent 鉴权、Dashboard 快照和配置通知会失败。

按钮完成后仍需执行：

1. 访问 Worker `/health`。
2. 核对 D1 database ID、管理员凭据和 `PAGES_ORIGIN`。
3. 在 Pages Production 设置 `REALTIME_URL`。
4. 重新部署 Pages。

Realtime Worker 必须具备：

| 类型 | 绑定/变量 | 值 |
|---|---|---|
| D1 | `DB` | 与 Pages 完全相同的生产 D1 |
| Durable Object | `VPS_PRESENCE` | 类 `VpsPresence` |
| Durable Object | `DASHBOARD_HUB` | 类 `DashboardHub` |
| Variable | `ADMIN_USERNAME` | 与 Pages 完全相同 |
| Variable | `PAGES_ORIGIN` | Pages 完整来源，例如 `https://your-kui.pages.dev`，末尾无 `/` |
| Secret | `ADMIN_PASSWORD` | 与 Pages 完全相同 |

两个 Durable Object 类必须是 SQLite-backed：

```text
VpsPresence
DashboardHub
```

Cloudflare 中可能显示为：

```text
kui-realtime_VpsPresence
kui-realtime_DashboardHub
```

这是正常命名空间前缀，不是重复 Worker，不要删除。

### 本地首次部署：使用 Wrangler 注册 Durable Object

不使用一键按钮时，首次本地部署必须让 Worker 代码、两个 Durable Object 绑定和 `new_sqlite_classes` migration 在同一次配置发布中生效。不要先在控制台手工创建同名但彼此独立的 Durable Object namespace。

前提：

- 本机已安装 Node.js LTS 和 npm。
- 已执行 `npx wrangler login`，或配置了对目标账户有 Workers、D1 和 Durable Objects 权限的 `CLOUDFLARE_API_TOKEN`。
- 建议记录 `npx wrangler --version`；本仓库没有锁定 Wrangler 版本。

先修改 `realtime/wrangler.jsonc`：

- `d1_databases[0].database_name`
- `d1_databases[0].database_id`
- `vars.ADMIN_USERNAME`
- `vars.PAGES_ORIGIN`

如果绑定现有 D1，先把真实 database ID 写入 `realtime/wrangler.jsonc`。如果希望 Wrangler 自动创建 `kui-db`，保留模板中省略 `database_id` 的写法。然后在仓库根目录执行首次部署，再写入 Secret：

```bash
npx wrangler deploy --config realtime/wrangler.jsonc
npx wrangler secret put ADMIN_PASSWORD --config realtime/wrangler.jsonc
```

首次 `deploy` 会注册两个 Durable Object 类并创建/绑定资源；此时管理员鉴权暂不可用。`secret put` 会发布包含 `ADMIN_PASSWORD` 的新 Worker 版本，完成后再执行 `/health` 和 Dashboard ticket 验证。

配置中已经声明首次 migration：

```json
{
  "tag": "v1",
  "new_sqlite_classes": ["VpsPresence", "DashboardHub"]
}
```

已经上线后不要删除 `v1`，也不要用同一个 tag 注册未来的新类。新增 DO 类时必须追加新 migration tag。

### 后续更新：Cloudflare 控制台复制代码

首次 Wrangler migration 成功后，可以只使用 Cloudflare 网页更新同一个 Worker：

1. 进入已部署的 `kui-realtime` Worker，不要另建同名 Worker。
2. 打开 **Edit code**。
3. 复制仓库 `realtime/src/index.js` 的完整内容，覆盖现有代码。
4. 确认现有绑定仍为 `DB`、`VPS_PRESENCE`、`DASHBOARD_HUB`。
5. 确认变量和 Secret 未被删除。
6. 保存并部署。

网页复制代码适用于不包含新 Durable Object 类或 migration 的普通代码更新。如果未来版本修改了 `realtime/wrangler.jsonc` 中的 migrations，必须再次使用 Wrangler 发布，不能只复制 JavaScript。

### 验证 Realtime Worker

访问：

```text
https://你的Worker域名/health
```

预期：

```json
{"ok":true,"service":"kui-realtime","version":1}
```

如果 `/health` 正常但 Dashboard WebSocket 失败，重点检查：

- Worker 和 Pages 是否绑定同一个 D1 database ID。
- Worker `ADMIN_USERNAME`、`ADMIN_PASSWORD` 是否与 Pages 完全相同。
- `PAGES_ORIGIN` 是否与浏览器 `location.origin` 完全一致。
- `VPS_PRESENCE`、`DASHBOARD_HUB` 绑定名是否拼写正确。
- `VpsPresence`、`DashboardHub` 类是否已通过 SQLite migration 注册。

## 第七步：启用实时模式

Worker 验证完成后：

1. 回到 Pages Production 环境变量。
2. 设置：

```text
REALTIME_URL=https://你的Realtime-Worker域名
```

3. 不要在末尾添加 `/`。
4. 重新部署 Pages。
5. 刷新 KUI 后台。

Pages `REALTIME_URL` 的优先级高于 D1 `sys_config.realtime_url`。推荐普通部署只使用 Pages 环境变量；如果同时设置，D1 值不能覆盖 Pages 变量。

成功后浏览器会先请求 `/dashboard/ticket`，再建立 `/dashboard/ws`。VPS 在下一次配置同步或服务重启后建立 `/agent/ws`。

## 第八步：添加 VPS 并执行 Full Deploy

### 添加服务器

1. 登录 KUI 后台。
2. 进入 **服务器与节点**。
3. 填写服务器别名、公网 IP 和系统类型。
4. 点击接入机器。
5. 刷新数据，确认服务器记录拥有独立 `agent_token`。
6. 在服务器卡片中选择 Debian/Ubuntu 或 Alpine，复制面板生成的 Full Deploy Command。

不要手工使用管理员密码或管理员密码哈希替代 Agent Token。

### Full Deploy 前提

- root Shell。
- Debian、Ubuntu 或 Alpine。
- `x86_64` 或 `aarch64`。
- 出站 HTTPS 可用。
- 住宅代理需要 TUN。
- VPS 公网 IP 必须先在面板登记。

### 命令结构示例

请以面板生成命令为准。以下仅展示结构：

```bash
# Debian / Ubuntu
apt-get update -y && apt-get install -y curl
bash <(curl -fsSL -H "Authorization: <AGENT_TOKEN>" \
  "https://<PAGES域名>/api/agent_update?ip=<VPS_IP>&component=full-installer") \
  --api "https://<PAGES域名>" \
  --ip "<VPS_IP>" \
  --token "<AGENT_TOKEN>"

# Alpine
apk update && apk add curl
curl -fsSL -H "Authorization: <AGENT_TOKEN>" \
  "https://<PAGES域名>/api/agent_update?ip=<VPS_IP>&component=full-installer" | \
  sh -s -- --api "https://<PAGES域名>" --ip "<VPS_IP>" --token "<AGENT_TOKEN>"
```

Full Deploy 会：

- 安装依赖。
- 下载固定版本 sing-box 并校验 SHA256。
- 安装 `/opt/kui/agent.py` 和 `/opt/kui/realtime_client.py`。
- 写入权限为 `600` 的 Agent 配置。
- 创建 `kui-agent` 和 `sing-box` 服务。
- 通过鉴权端点下载住宅安装器。
- 安装 `/opt/proxy_lite/lite_manager.py`、`proxy_server.py`、`realtime_client.py`。
- 创建 `proxy-lite` 服务并管理 `tun_main`、`tun_backup`。

Full Deploy 是系统级安装，会修改服务、网络配置和工作目录。请先在非关键 VPS 验证。

## 第九步：生产验收

### Debian/Ubuntu

```bash
systemctl is-active kui-agent sing-box proxy-lite
journalctl -u kui-agent -n 100 --no-pager
journalctl -u proxy-lite -n 100 --no-pager
stat -c '%a %n' /opt/kui/config.json /etc/proxy-lite/env
ss -tpn | grep python3
```

### Alpine

```bash
rc-service kui-agent status
rc-service sing-box status
rc-service proxy-lite status
tail -n 100 /var/log/kui-agent.log
tail -n 100 /var/log/proxy-lite.log
stat -c '%a %n' /opt/kui/config.json /etc/proxy-lite/env
ss -tpn | grep python3
```

启用住宅实时模式时，每台 VPS 正常应看到两条 Python 到 Cloudflare `443` 的长连接：Core 和 Proxy 各一条。

住宅隧道依赖外部 VPNGate 节点、TUN、网络质量和启发式出口检查。安装成功不保证立即达到 `2 / 2`，也不保证所有出口一定是住宅网络。可能出现 `0 / 2`、`1 / 2` 或候选耗尽，应结合日志排查。

### 浏览器验收

管理员后台打开时：

- `/dashboard/ticket` 成功。
- `/dashboard/ws` 显示 WebSocket 已连接。
- CPU、内存、网速通常 1-5 秒更新。
- 主备、出口、上下线和配置结果关键变化立即推送。
- WebSocket 健康时不再周期请求 `/api/data`、`/api/probe/public`、`/api/proxy/nodes`、`/api/ui_ping`。
- 断线持续满 30 秒后才启动 HTTP fallback。

# 实时与 fallback 行为

## WebSocket 健康

| 通道 | 周期/行为 |
|---|---|
| Core 状态 | 每 5 秒发送到 `VpsPresence` |
| Proxy 状态 | 每 5 秒发送到 `VpsPresence` |
| Dashboard 普通快照 | Core 的 5 秒状态驱动 Core+Proxy 合并快照 |
| Proxy 关键变化 | 在下一次 Proxy 状态帧中推送，通常不超过 5 秒；配置结果处理完成后立即推送 |
| 配置通知和结果 | 立即推送 |
| HTTP 状态持久化 | 约每 15 分钟 |
| HTTP 配置权威校验 | 约每 15 分钟 |

`VpsPresence` 在 Durable Object storage 中保存一个复合 `state` 对象，并使用 `boot_id` 和单调 `seq` 丢弃重复或乱序消息。DashboardHub 不重复持久化每个 VPS 快照。

## WebSocket 断开

Core 和 Proxy Agent 的 WebSocket 连续不可用时，前 30 秒只重连，不启动高频 HTTP。浏览器如果已经建立过健康 Dashboard WebSocket，断线后也等待 30 秒再恢复轮询；首次登录或首次启用一个不可用的 `REALTIME_URL` 时，登录流程会先启动 HTTP polling，因此不会等待 30 秒才获得数据。

连续断开满 30 秒后：

| 组件 | HTTP fallback |
|---|---|
| Core 状态 | 通常 90-300 秒，受服务端 interval 限制 |
| Core 配置 | UI 活跃时可为 30 秒，否则 300 秒 |
| Proxy 状态 | 90 秒 |
| Proxy 配置 | 300 秒 |
| 浏览器主数据 | 最低 15 秒 |
| 浏览器探针 | 最低 30 秒 |
| 浏览器 UI ping | 最低 60 秒 |

WebSocket 恢复后自动停止高频 HTTP fallback。

# 请求量和容量

以下是按当前实现推算的工作负载，不是 Cloudflare 永久额度承诺。Cloudflare 定价和免费额度可能变化，请以目标账户 Dashboard 和官方文档为准。

以 2 台 VPS、每台运行 Core + Proxy、1 个常开管理员后台为例：

- Agent WebSocket 状态消息约 69,120 条/天。
- 按 WebSocket 入站消息 20:1 的 Durable Objects 请求计费折算，Agent 入站约 3,456 个 DO 计费请求/天。
- Core 驱动 Presence → Hub 更新约 34,560 个 DO 计费请求/天。
- 加上 Dashboard 应用层 ping、配置结果、连接和重连后，稳态约 3.8-3.9 万个 DO 计费请求/天。
- Presence 对 SQLite-backed DO storage 的单 `state` 写入稳态约 6.9 万行/天，配置结果和连接事件会增加少量写入。
- 两台 VPS 的 Pages Functions 健康态请求约 1,200/天，主要来自 15 分钟 HTTP 检查点和每小时组件更新检查。
- Dashboard WebSocket 健康时没有周期 Pages API 轮询，但仍存在 WebSocket 消息、本地 UI timer、显式用户操作和重连请求。

Realtime Worker 的 `/notify`、snapshot 和 activity fan-out 当前最多处理前 100 台服务器。超过 100 台需要先修改分片和 fan-out 设计。

# 住宅代理说明

Full Deploy 默认安装住宅代理管理器。它通过公开 VPNGate OpenVPN 节点建立主备隧道，并使用外部检测结果做启发式筛选。

### 端口和防火墙

- 默认监听端口为 `7920/TCP`，实际端口以住宅页面配置为准。
- 服务支持 SOCKS5 和 HTTP proxy。
- 安装器不会替你安全开放公网代理端口。
- 外部设备需要访问时，应同时配置云防火墙和系统防火墙，只允许可信来源 IP。
- Core Agent 上报不需要入站端口，只需要出站 HTTPS；住宅代理对外服务是另一回事。

### 验证代理出口

请替换为真实凭据和实际端口，避免将长期密码留在共享 Shell history：

```bash
export PROXY_USER='你的用户名'
export PROXY_PASS='你的密码'
export PROXY_PORT='7920'

curl -fsS --max-time 30 \
  --proxy "socks5h://127.0.0.1:${PROXY_PORT}" \
  --proxy-user "${PROXY_USER}:${PROXY_PASS}" \
  https://api.ipify.org
```

返回值应是当前 ACTIVE 隧道出口，而不是 VPS 本机公网 IP。

### 单独修复住宅组件

标准部署不需要第二条住宅命令。仅当 Core 正常、住宅组件需要单独修复时，使用该 VPS 当前 Agent Token：

```bash
# Debian / Ubuntu
bash <(curl -fsSL -H "Authorization: <AGENT_TOKEN>" \
  "https://<PAGES域名>/api/agent_update?ip=<VPS_IP>&component=proxy-installer") \
  --domain "https://<PAGES域名>" --controller "https://<PAGES域名>" \
  --ip "<VPS_IP>" --token "<AGENT_TOKEN>"

# Alpine
apk add --no-cache bash curl
curl -fsSL -H "Authorization: <AGENT_TOKEN>" \
  "https://<PAGES域名>/api/agent_update?ip=<VPS_IP>&component=proxy-installer" | \
  bash -s -- --domain "https://<PAGES域名>" --controller "https://<PAGES域名>" \
  --ip "<VPS_IP>" --token "<AGENT_TOKEN>"
```

# 更新与发布

## GitHub 和 Pages

推送 GitHub 不等于生产已更新。必须满足：

1. 推送到 Pages 配置的 Production branch。
2. Pages Production deployment 成功。
3. `/api/agent_update` 已返回新组件和新的 `X-Agent-SHA256`。

## VPS 自动更新

Core Agent 每小时检查：

```text
/opt/kui/agent.py
/opt/kui/realtime_client.py
```

Proxy Manager 每小时检查：

```text
/opt/proxy_lite/lite_manager.py
/opt/proxy_lite/proxy_server.py
/opt/proxy_lite/realtime_client.py
```

更新流程：限制体积 → 校验 `X-Agent-SHA256` → `py_compile` → 原子替换 → 自重载。下载、校验或编译失败时保留现有组件。

Agent 不直接从 GitHub Raw 更新，而是使用 Pages 的鉴权 `/api/agent_update`。公开 `/vps/*.py` 可能受 CDN 缓存影响，不应用于判断 Agent 是否已发布最新版本。

# Telegram 和离线检查

Pages Functions 不会自动获得 Cron Trigger。需要离线告警时，使用 Cloudflare Worker Cron、UptimeRobot 或其他调度器定期调用：

```http
POST https://你的Pages域名/api/cron_check
Authorization: Bearer <CRON_SECRET>
```

当前 `/api/cron_check` 只检查 D1 中 Core HTTP 的 `servers.last_report`，离线阈值约 6 分钟。Realtime 健康时该字段约每 15 分钟才持久化一次，因此直接启用旧 Cron 检查会对健康 WebSocket Agent 产生误报。Realtime 模式下不要把该接口作为可靠离线告警，除非先修改后端检查 Presence，或把 HTTP 持久化间隔缩短到低于告警阈值。Agent 再次 HTTP 上报会重置告警标记，但当前代码没有恢复在线通知。

Telegram Bot Token 和 Chat ID 可以在探针后台写入 D1。当前代码在存在对应 D1 行时会覆盖 Pages 的 `TG_BOT_TOKEN`、`TG_CHAT_ID`，即使 D1 值为空；因此只想使用 Pages Secret 时不要保存空的 D1 Telegram 配置。使用 Telegram Webhook 命令时必须配置 `TG_WEBHOOK_SECRET`。

# 历史 Agent 迁移

`LEGACY_AGENT_AUTH=true` 仅用于旧 Agent 临时迁移。它允许旧 Agent 使用管理员密码 SHA256，而不是服务器独立 Token，风险明显高于当前模式。

- 新部署不要启用。
- 旧 Agent 获取新配置并保存 `agent_token` 后立即关闭。
- 最迟在 `2026-08-01T00:00:00Z` 后代码会拒绝该兼容模式。
- 最稳妥方式是逐台重新执行当前面板生成的 Full Deploy Command。

# 高级自定义安全提示

后台的自定义 `<head>` 和底部 Script 功能会执行管理员提供的任意 HTML/JavaScript。它们没有沙箱，也不是经过 Vue 安全净化的内容。

- 只允许完全可信的管理员使用。
- 不要粘贴来源不明的主题脚本。
- 发现异常跳转、凭据读取或网络请求时，立即清空自定义代码并轮换管理员凭据。

# 故障排查

## Pages 提示缺少 DB

- 检查 Pages Production 环境是否绑定 D1。
- 绑定名必须是 `DB`。
- 修改后重新部署 Production。
- Preview 绑定不会自动作用于 Production。

## `/api/agent_update` 返回 503

- 如果提示 `ASSETS binding is unavailable`，确认主应用部署为 Cloudflare Pages，而不是普通 Worker。
- `/api/agent_update` 不会因为缺少 `PROXY_USER` 或 `PROXY_PASS` 返回 `503`。如果 `/api/proxy/config`、`/api/proxy/nodes` 或住宅组件配置请求提示缺少代理凭据，再配置它们并重新部署 Pages。

## Agent 返回 401/403

- VPS IP 必须已在面板登记。
- `Authorization` 必须是该服务器自己的 `agent_token`。
- 不要使用管理员密码、管理员哈希或其他 VPS Token。
- 重新复制面板当前生成的 Full Deploy Command。

## Worker `/health` 正常但 WebSocket 失败

- 检查 `PAGES_ORIGIN` 是否完全一致。
- 检查 Pages 和 Worker 管理员凭据是否一致。
- 检查 Worker 和 Pages 是否绑定同一个 D1 ID。
- 检查两个 DO 绑定名和类名。
- 检查浏览器 Network 中 `/dashboard/ticket` 和 `/dashboard/ws`。
- 检查 VPS 是否能出站访问 Worker `443`。

## 探针不更新

```bash
systemctl status kui-agent
journalctl -u kui-agent -n 150 --no-pager
cat /opt/kui/config.json
```

Alpine 使用 `rc-service` 和 `/var/log/kui-agent.log`。不要把包含 Token 的完整配置发到公开渠道。

## 住宅代理显示 `0 / 2` 或 `1 / 2`

- 检查 `proxy-lite` 服务和日志。
- 检查 `/dev/net/tun`。
- 检查 OpenVPN 进程和 `tun_main`、`tun_backup`。
- 检查 Pages 是否配置 `PROXY_USER`、`PROXY_PASS`；缺失通常返回 `503`。
- 检查 `/api/proxy/config` 是否为 `401/403`。
- 检查 VPS 到 VPNGate、ipify、testisp 和连通性检测目标的出站网络。
- 所选国家可能暂时没有可用候选，或候选被数据中心筛选规则拒绝。

## GitHub 已推送但 VPS 没更新

- 确认推送到 Production branch。
- 确认 Pages 生产发布完成。
- 新 Agent 最多每小时检查一次。
- 旧 Agent 没有新热更新逻辑时，需要重新执行 Full Deploy 完成引导。

# 项目结构

```text
K-UI-main/
├── index.html
├── functions/
│   └── api/
│       └── [[path]].js
├── realtime/
│   ├── src/
│   │   └── index.js
│   └── wrangler.jsonc
└── vps/
    ├── agent.py
    ├── realtime_client.py
    ├── kui.sh
    ├── lite_manager.py
    ├── proxy_server.py
    └── residential-proxy.sh
```

# 技术栈

| 层级 | 技术 |
|---|---|
| 前端 | Vue 3、Tailwind CSS、ECharts、Chart.js、Leaflet |
| 主后端 | Cloudflare Pages Functions |
| 实时后端 | Cloudflare Worker、Hibernation WebSocket、Durable Objects |
| 数据库 | Cloudflare D1 |
| 代理核心 | sing-box、OpenVPN |
| VPS 服务 | Python 3、systemd/OpenRC |

# 贡献与支持

- Issues：https://github.com/a6216abcd/K-UI/issues
- Pull Requests：https://github.com/a6216abcd/K-UI/pulls

提交 Issue 时不要附带管理员密码、Agent Token、Cloudflare API Token、住宅代理凭据或完整 VPS 配置。

本仓库当前未包含独立 `LICENSE` 文件，请在复制、分发或商业使用前自行确认上游项目和各依赖的许可要求。

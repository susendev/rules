# Claude / Anthropic 扩展规则调研

> 核对日期：2026-07-28。最终口径为“宁可错杀，不可遗漏”，允许共享基础设施、历史端点、IP 段和 ASN 被一并送入 `🤖 AI 服务`。

## 结论

单独使用 MetaCubeX `anthropic.yaml` 不完整。它的 8 条规则能覆盖主要 Anthropic 自有域名，但缺少 `claude.app`、官方固定 IP、Claude Code 的部分更新/遥测主机、Office/第三方云依赖、MCP 根域和网站共享资源。

当前组合改为四层：

```ini
ruleset=🤖 AI 服务,clash-domain:https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/anthropic.yaml
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/susendev/rules/refs/heads/master/rules/claude-ext.list
ruleset=🤖 AI 服务,clash-classic:https://raw.githubusercontent.com/VPSDance/ai-proxy-rules/refs/heads/main/rules/clash/anthropic.yaml
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/cutethotw/ClashRule/refs/heads/main/Rule/Claude.list
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Ruleset/AI.list
```

- MetaCubeX：维护精确的 Anthropic 自有域名基础表。
- 本地 ext：补官方最新域名、共享依赖、网站资源与固定 IP。
- VPSDance / cutethotw：补宽泛 SaaS、历史端点、Google 资源、IP/ASN 和关键词兜底。
- ACL4SSR：继续提供 `DOMAIN-KEYWORD,anthropic|claude` 和综合 AI 兜底。

## 调研来源

### 官方来源

- [Claude Code：Enterprise network configuration](https://code.claude.com/docs/en/network-config)
- [Claude Code Desktop：Network access requirements](https://code.claude.com/docs/en/desktop#network-access-requirements)
- [Claude Code on the web：Default allowed domains](https://code.claude.com/docs/en/claude-code-on-the-web#default-allowed-domains)
- [Claude Platform：官方入站/出站 IP 范围](https://platform.claude.com/docs/en/api/ip-addresses)
- [Claude for Microsoft 365：Network allowlist](https://support.claude.com/en/articles/13945233-use-claude-for-microsoft-365-with-third-party-platforms)
- [Model Context Protocol 官方文档](https://modelcontextprotocol.io)
- `claude.com` 与 `anthropic.com` 的 2026-07-28 实时页面资源

### GitHub 来源

- [MetaCubeX：anthropic.yaml 固定快照](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/anthropic.yaml)
- [ACL4SSR：AI.list 固定快照](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)
- [Blackmatrix7：Claude.list](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/Claude/Claude.list)
- [VPSDance：anthropic.yaml 固定快照](https://github.com/VPSDance/ai-proxy-rules/blob/e256f87e8522b829071a8c13c26a96819ca69703/rules/clash/anthropic.yaml)
- [VPSDance：Anthropic 来源与人工补丁](https://github.com/VPSDance/ai-proxy-rules/blob/e256f87e8522b829071a8c13c26a96819ca69703/data/sources/anthropic.yaml)
- [cutethotw：Claude.list 固定快照](https://github.com/cutethotw/ClashRule/blob/637fa05ff8c41bb8f4031f5db64bbea2e16aa1d6/Rule/Claude.list)
- [MCP 官方 GitHub 组织](https://github.com/modelcontextprotocol)

用户同时提供的 [Blackmatrix7 OpenAI.list](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/OpenAI/OpenAI.list) 属于 OpenAI 来源，已在 `docs/openai-claude-rules-research.md` 中处理。

## MetaCubeX 已覆盖的 8 条

```yaml
servd-anthropic-website.b-cdn.net
+.anthropic.com
+.clau.de
+.claude.ai
+.claude.com
+.claudemcpclient.com
+.claudemcpcontent.com
+.claudeusercontent.com
```

它们已经覆盖：

- `api.anthropic.com`、`a-api.anthropic.com`、`assets-proxy.anthropic.com`
- `claude.ai`、`downloads.claude.ai`、`assets.claude.ai`
- `claude.com`、`platform.claude.com`、`code.claude.com`
- `mcp-proxy.anthropic.com`
- `bridge.claudeusercontent.com`、Artifact 和用户内容域名
- Claude MCP client / content 域名

## 本地 ext 的 22 条

```text
DOMAIN-SUFFIX,claude.app
DOMAIN-SUFFIX,anthropic.com.cn
DOMAIN-SUFFIX,modelcontextprotocol.io
DOMAIN-SUFFIX,sanity.io
DOMAIN,4zrzovbb.api.sanity.io
DOMAIN,cdn.prod.website-files.com
DOMAIN,d3e54v103j8qbb.cloudfront.net
DOMAIN,http-intake.logs.us5.datadoghq.com
DOMAIN,browser-intake-us5-datadoghq.com
DOMAIN,o1158394.ingest.us.sentry.io
DOMAIN,cdn.usefathom.com
DOMAIN,raw.githubusercontent.com
DOMAIN,formulae.brew.sh
DOMAIN,registry.npmjs.org
DOMAIN,appsforoffice.microsoft.com
DOMAIN,login.microsoftonline.com
DOMAIN,accounts.google.com
DOMAIN-SUFFIX,amazonaws.com
DOMAIN-SUFFIX,googleapis.com
DOMAIN-SUFFIX,services.ai.azure.com
IP-CIDR,160.79.104.0/21,no-resolve
IP-CIDR6,2607:6bc0::/48,no-resolve
```

### 第一方与官方固定网络

- `claude.app`：Claude Desktop 官方要求，并覆盖动态 `*.livepreview.claude.app`。
- `160.79.104.0/21`：包含 Anthropic 官方入站 `/23`，同时等于官方出站 IPv4 范围。
- `2607:6bc0::/48`：Anthropic 官方入站 IPv6 范围。
- `anthropic.com.cn`：VPSDance 收录但当前不解析；按“不可遗漏”保留未来/历史覆盖。

### Claude Code 与 MCP

- 两个 Datadog 主机：Claude Code 官方可选遥测和错误报告。
- `raw.githubusercontent.com`：Release Notes。
- `formulae.brew.sh`：Homebrew 版本检查。
- `registry.npmjs.org`：npm / bun 安装 Claude Code 时使用的包注册表。
- `modelcontextprotocol.io`：Anthropic 创建并已捐赠给 AAIF 的 MCP 官方文档和注册体系。
- `storage.googleapis.com` 已被本地 `DOMAIN-SUFFIX,googleapis.com` 覆盖。

### Claude for Microsoft 365 与第三方云

- Office / Entra：`appsforoffice.microsoft.com`、`login.microsoftonline.com`。
- Amazon Bedrock：`DOMAIN-SUFFIX,amazonaws.com`，覆盖 STS 与区域 Bedrock Runtime。
- Google Vertex AI：`accounts.google.com`、`DOMAIN-SUFFIX,googleapis.com`。
- Azure Foundry：`DOMAIN-SUFFIX,services.ai.azure.com`。
- 专用 Sentry：`o1158394.ingest.us.sentry.io`。
- 用户自定义 LLM Gateway 无固定域名，无法预先穷举。

### 网站和共享资源

- 当前 `claude.com` 使用 Claude 专属 Sanity project host 及 Sanity CDN。
- 当前 `anthropic.com` 使用 Webflow CDN 与 CloudFront 公共脚本。
- Blackmatrix7 Claude 表收录 `cdn.usefathom.com`。

## VPSDance 与 cutethotw 的额外宽覆盖

两张社区表继续补充：

- `anthropic.auth0.com`、`anthropic-com.ghost.io`、`anthropic.com.cdn.cloudflare.net`
- GrowthBook、Intercom、Sentry、Statsig、Sift、Datadog 等共享服务
- Google `gstatic.com` 静态资源与 `storage.googleapis.com`
- Anthropic IPv4/IPv6、`IP-ASN,399358`
- `DOMAIN-KEYWORD,anthropic|claude|datadog|sentry|sift`

VPSDance Anthropic YAML 是混合 Clash provider，使用 Subconverter 文档规定的 `clash-classic:` 类型。cutethotw 是普通 classical 文本表，沿用本项目现有的 URL 规则集读取方式。

## “宁可错杀”的具体副作用

- `DOMAIN-SUFFIX,amazonaws.com`、`googleapis.com`、`sanity.io` 会把大量非 Claude 流量送入 AI。
- 社区表的 `sentry.io`、Intercom、Statsig、Auth0、ASN 和关键词规则也会匹配其它租户。
- `raw.githubusercontent.com`、Homebrew、Office.js、Webflow、CloudFront 不是 Claude 专属服务。
- cutethotw 与 VPSDance 存在大量重复规则，生成后的规则数量会增加。

这些行为不是疏漏，而是用户明确选择覆盖优先后的已接受结果。

## 完整性边界

当前组合覆盖 Claude Web、API、Console、Desktop、Code、Chrome、MCP、Office、第三方云、下载、网站资源、遥测、官方 IP 与社区历史端点。

仍然无法通过固定规则预先穷举：

- 用户自定义 `ANTHROPIC_BASE_URL` 或 LLM Gateway
- 用户自行配置的任意 MCP server
- Claude 浏览器功能访问的任意目标网站
- 未来新增或动态调整的域名、IP 与供应商

维护时应先检查两份 Anthropic 官方网络白名单和 IP 页面，再比较 MetaCubeX、VPSDance、cutethotw 与 ACL4SSR 的最新提交。

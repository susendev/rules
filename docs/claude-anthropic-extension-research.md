# Claude / Anthropic 扩展规则调研

## 结论

截至 2026-07-28，MetaCubeX 的 `anthropic.yaml` 已覆盖 Claude Web、Anthropic API、Console、Claude Code、Chrome Bridge、MCP 与用户内容域名中的绝大多数 Anthropic 自有域名，但并不完整：

- Claude Desktop 官方白名单已包含 `claude.app` / `*.claude.app`，上游尚未收录。
- Claude Code 官方网络要求列出了两个可选 Datadog 遥测端点，上游无法通过 Anthropic 自有域名后缀覆盖。
- Claude for Microsoft 365 官方文档列出了一个专用 Sentry 端点。
- 当前 `claude.com` 页面使用一个 Claude 专属的 Sanity 项目 API 域名。
- Blackmatrix7 的 Claude 专项规则长期收录 `cdn.usefathom.com`。

因此新增 `rules/claude-ext.list`，补充上游没有覆盖、且能确认与 Claude 相关的 6 条规则。配置仍以 MetaCubeX 为基础表，扩展表紧随其后，最后保留 ACL4SSR 的综合 AI 规则作为兜底。

## 调研来源

### 官方来源

- [Claude Code：Enterprise network configuration](https://code.claude.com/docs/en/network-config)
- [Claude Code：Desktop application / Network access requirements](https://code.claude.com/docs/en/desktop#network-access-requirements)
- [Claude Code on the web：Default allowed domains](https://code.claude.com/docs/en/claude-code-on-the-web#default-allowed-domains)
- [Claude for Microsoft 365：Use Claude for Microsoft 365 with third-party platforms](https://support.claude.com/en/articles/13945233-use-claude-for-microsoft-365-with-third-party-platforms)
- `claude.com` 与 `anthropic.com` 的 2026-07-28 实时页面资源引用

### GitHub 规则来源

- [MetaCubeX / meta-rules-dat：anthropic.yaml](https://github.com/MetaCubeX/meta-rules-dat/blob/meta/geo/geosite/anthropic.yaml)
- [Blackmatrix7 / ios_rule_script：Claude.list](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/Claude/Claude.list)
- [V2Fly / domain-list-community：Anthropic / Claude 域名讨论](https://github.com/v2fly/domain-list-community/issues/2860)

## 现有上游覆盖

MetaCubeX 当前的 Anthropic domain provider 有 8 条：

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

这些后缀已经覆盖：

| 官方域名或用途 | 上游匹配项 | 结果 |
| --- | --- | --- |
| `api.anthropic.com`、`a-api.anthropic.com` | `+.anthropic.com` | 已覆盖 |
| `statsig.anthropic.com`、`assets-proxy.anthropic.com` | `+.anthropic.com` | 已覆盖 |
| `claude.ai`、`downloads.claude.ai`、`assets.claude.ai` | `+.claude.ai` | 已覆盖 |
| `*.livepreview.claude.ai` | `+.claude.ai` | 已覆盖 |
| `claude.com`、`platform.claude.com`、`code.claude.com` | `+.claude.com` | 已覆盖 |
| `mcp-proxy.anthropic.com` | `+.anthropic.com` | 已覆盖 |
| `bridge.claudeusercontent.com`、用户内容与 Artifact | `+.claudeusercontent.com` | 已覆盖 |
| Claude MCP client / content | 对应两个 `claudemcp*` 后缀 | 已覆盖 |
| `servd-anthropic-website.b-cdn.net` | 精确规则 | 已覆盖 |

因此不在扩展文件中重复这些规则。

## 新增扩展规则

```text
DOMAIN-SUFFIX,claude.app
DOMAIN,4zrzovbb.api.sanity.io
DOMAIN,http-intake.logs.us5.datadoghq.com
DOMAIN,browser-intake-us5-datadoghq.com
DOMAIN,o1158394.ingest.us.sentry.io
DOMAIN,cdn.usefathom.com
```

逐项依据：

| 扩展规则 | 用途与依据 | 级别 |
| --- | --- | --- |
| `DOMAIN-SUFFIX,claude.app` | Claude Desktop 官方要求 `claude.app`、`*.claude.app`，并明确存在动态生成的 `*.livepreview.claude.app` | 官方、必补 |
| `DOMAIN,4zrzovbb.api.sanity.io` | 2026-07-28 的 `claude.com` 页面直接引用该 Claude 专属 Sanity project API | 实时页面、精确项目域名 |
| `DOMAIN,http-intake.logs.us5.datadoghq.com` | Claude Code 直接使用 Anthropic API 时的可选运行遥测 | 官方、可选流量 |
| `DOMAIN,browser-intake-us5-datadoghq.com` | Claude Code 经灰度开关启用的可选错误报告 | 官方、可选流量 |
| `DOMAIN,o1158394.ingest.us.sentry.io` | Claude for Microsoft 365 崩溃与错误报告 | 官方、可选流量 |
| `DOMAIN,cdn.usefathom.com` | Blackmatrix7 Claude 专项规则收录的网站统计端点 | GitHub 社区补充、可选流量 |

其中四个遥测/统计域名不是模型请求的必要条件，但纳入官方明确列出或 GitHub Claude 专项表长期维护的完整主机名，可以避免已知 Claude 相关请求落到 `🐟 漏网之鱼`。不使用 `DOMAIN-SUFFIX,datadoghq.com`、`DOMAIN-SUFFIX,sentry.io` 等更宽泛规则。

## 共享域名的取舍

Datadog 的区域 intake 主机和 `cdn.usefathom.com` 本身仍可能服务其它租户，Clash 的域名规则无法再按租户或 URL path 细分。考虑到本次要求是“尽可能完整不遗漏”，这里接受少量可选遥测流量也进入 `🤖 AI 服务` 的副作用：

- 两个 Datadog 主机来自 Claude Code 当前官方网络要求。
- Fathom 主机来自仍在发布的 Blackmatrix7 Claude 专项规则，但没有 Anthropic 官方归属证明，可信度低于其它 5 条。
- 这些规则只匹配完整主机名，没有扩大到供应商的整个根域名。

如果未来更看重“只允许 Claude 专属域名”而不是“尽可能完整”，可以先移除 `cdn.usefathom.com`，再移除两个可选 Datadog 端点，不影响 Claude 的核心模型请求。

## 刻意不加入的高范围共享域名

以下域名确实出现在官方网络要求或实时页面中，但它们承载大量非 Claude 的核心下载、内容或平台流量，影响范围明显大于可选遥测主机：

| 域名 | Claude 用途 | 不加入原因 |
| --- | --- | --- |
| `storage.googleapis.com` | 旧版安装/更新、插件元数据、Artifact 上传 | Google Cloud Storage 全局共享；现有 Google 规则可处理 |
| `raw.githubusercontent.com` | Claude Code 更新后的 Release Notes | GitHub 全局共享；不应把所有 Raw 内容归入 AI |
| `formulae.brew.sh` | Homebrew 版本检查 | Homebrew 全局共享 |
| `appsforoffice.microsoft.com` | Microsoft 365 Office Runtime | Microsoft 全局共享；已有 Microsoft 策略组 |
| `cdn.sanity.io`、`api.sanity.io` | `claude.com` 的 CMS 内容与图片 | Sanity 全局共享；仅收录 Claude 专属 project host |
| `cdn.prod.website-files.com` | `anthropic.com` 的 Webflow 静态资源 | Webflow 全局共享 |
| `d3e54v103j8qbb.cloudfront.net` | Webflow 公共脚本/CDN | 多网站共享 CloudFront 分发 |
| Microsoft、AWS、Google Cloud、Azure 的第三方连接器域名 | 用户主动连接的数据源 | 应由对应服务策略处理，不属于 Claude 自有服务 |

这些共享域名即使不进入 `🤖 AI 服务`，也会继续由后续 Google、Microsoft、国外服务或 `🐟 漏网之鱼` 策略处理，不会被拦截。

## 配置顺序

```ini
ruleset=🤖 AI 服务,clash-domain:https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/anthropic.yaml
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/susendev/rules/refs/heads/master/rules/claude-ext.list
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Ruleset/AI.list
```

顺序含义：

1. MetaCubeX 持续接收上游新增的 Anthropic 自有域名。
2. 本仓库扩展表补上已确认但上游遗漏的精确规则。
3. ACL4SSR 综合 AI 表处理其它 AI 服务和兼容性兜底。

## 完整性边界与维护建议

- 当前组合为 8 条上游 Anthropic provider 规则加 6 条本地扩展规则，并有 ACL4SSR AI 综合表兜底。
- 这已经覆盖本次查询到的 Claude Web、Anthropic API、Console、Claude Desktop、Claude Code、Claude in Chrome、MCP、Microsoft 365、用户内容、下载与专用遥测端点。
- 第三方云模型入口（Amazon Bedrock、Google Cloud、Microsoft Foundry）、用户自定义 `ANTHROPIC_BASE_URL`、任意 MCP 服务和 Claude 浏览器访问的目标网站不属于固定 Claude 域名，不能也不应由这份规则穷举。
- Anthropic 会调整 CDN、遥测供应商与动态子域名，无法对未来域名作永久保证。后续复查优先比较 Claude Code 的两份官方网络白名单，再检查 MetaCubeX 和 Blackmatrix7 的更新。

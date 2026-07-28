# OpenAI / ChatGPT 与 Claude / Anthropic 分流规则研究

> 核对日期：2026-07-28。最终取舍为“宁可错杀，不可遗漏”：优先覆盖 OpenAI / ChatGPT 与 Claude / Anthropic 已知的第一方域名、第三方依赖、IP 段与 ASN，接受共享基础设施被一并送入 AI 策略，并兼容传统 Clash 规则类型。

## 结论

当前使用三层组合：MetaCubeX 精准基础表、本仓库官方补充表、GitHub 宽覆盖表与 ACL4SSR 综合兜底。

```ini
# OpenAI / ChatGPT
ruleset=🤖 AI 服务,clash-domain:https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/openai.yaml
ruleset=🤖 AI 服务,[]DOMAIN-KEYWORD,chatgpt-async-webps-prod
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/susendev/rules/refs/heads/master/rules/openai-ext.list
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/OpenAI/OpenAI.list

# Claude / Anthropic
ruleset=🤖 AI 服务,clash-domain:https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/anthropic.yaml
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/susendev/rules/refs/heads/master/rules/claude-ext.list
ruleset=🤖 AI 服务,clash-classic:https://raw.githubusercontent.com/VPSDance/ai-proxy-rules/refs/heads/main/rules/clash/anthropic.yaml
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/cutethotw/ClashRule/refs/heads/main/Rule/Claude.list

# 综合 AI 兜底
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Ruleset/AI.list
```

理由：

- MetaCubeX 的 domain provider 输出来自 v2fly 分类，OpenAI 有 22 条标准域名、Anthropic 有 8 条，第一方域名和专用 CDN/Azure 主机覆盖明显比 ACL4SSR 单表更完整，同时避免对整个 Azure、Google、Cloudflare 后缀做代理。[OpenAI 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/openai.yaml)；[Anthropic 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/anthropic.yaml)；[MetaCubeX 构建流程](https://github.com/MetaCubeX/meta-rules-dat/blob/4178770badecb1b349fbcd62c737e0d7a2079729/.github/workflows/run.yml#L40-L50)
- v2fly 的第 23 条 OpenAI 规则是动态 Azure Web PubSub 正则；MetaCubeX classical 将其输出为 `DOMAIN-REGEX`，但传统 Clash 不支持该类型。本项目使用 VPSDance 的较宽关键词 `chatgpt-async-webps-prod` 等价补齐，避免遗漏未来变体。[v2fly 固定源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/openai)；[MetaCubeX classical 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/classical/openai.list)
- `clash-domain:` 是 Subconverter 明确支持的规则集类型，会把 domain provider 转成标准 `DOMAIN` / `DOMAIN-SUFFIX` 规则，适合本项目当前的规则生成流程。[Subconverter 固定版本文档](https://github.com/tindy2013/subconverter/blob/a0d4eab28cb8b6c782d4ce5c3a918de4829b4a72/README-cn.md#L889-L906)；[类型定义](https://github.com/tindy2013/subconverter/blob/a0d4eab28cb8b6c782d4ce5c3a918de4829b4a72/src/handler/settings.cpp#L22-L23)
- ACL4SSR 仍有价值，因为它是一张 47 条的多平台 AI 聚合表；但其中的关键词和共享 SaaS 后缀不够精确，应作为完整性兜底，而不是 OpenAI/Claude 的唯一权威来源。[ACL4SSR 固定源码](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)
- 最终要求明确为“宁可错杀，不可遗漏”，因此接受 Blackmatrix7、VPSDance 与 cutethotw 表中的宽泛 SaaS 后缀、IP 和 ASN；重复项全部指向同一个 `🤖 AI 服务`，不会改变策略结果。VPSDance OpenAI 中传统 Clash 不支持的 `DOMAIN-REGEX` 继续由 `DOMAIN-KEYWORD,chatgpt-async-webps-prod` 等价覆盖，其余新增网络规则归入本地 ext。共享基础设施被误分流是已接受的代价。[blackmatrix7 OpenAI](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/OpenAI/OpenAI.list)；[VPSDance OpenAI](https://github.com/VPSDance/ai-proxy-rules/blob/main/rules/clash/openai.yaml)；[VPSDance Anthropic](https://github.com/VPSDance/ai-proxy-rules/blob/main/rules/clash/anthropic.yaml)；[cutethotw Claude](https://github.com/cutethotw/ClashRule/blob/main/Rule/Claude.list)

## 来源对比

| 来源 | OpenAI / ChatGPT 覆盖 | Claude / Anthropic 覆盖 | 更新情况 | 判断 |
| --- | --- | --- | --- | --- |
| ACL4SSR `AI.list` | `openai.com`、`chatgpt.com`、`oaistatic.com`、`oaiusercontent.com`、`sora.com`，另有 `DOMAIN-KEYWORD,openai` | `anthropic.com`、`claude.ai`、`claude.com`、`claudeusercontent.com`，另有 `anthropic`/`claude` 关键词 | 文件最近一次提交为 [2026-01-25](https://github.com/ACL4SSR/ACL4SSR/commit/591b05e72d2d6c17d18a64e9af0c43e9b237b152) | 多 AI 平台聚合方便，但缺少部分新域名和高级语音主机，且关键词/共享后缀偏宽 |
| blackmatrix7 OpenAI | 35 条，包含第一方域名、LiveKit、Azure、监控/登录/支付第三方以及 IP/ASN | — | 文件头标注 `UPDATED: 2025-06-06`；[固定源码](https://github.com/blackmatrix7/ios_rule_script/blob/8818705adee20571a856daf11c9fc69c4929109a/rule/Clash/OpenAI/OpenAI.list) | 表面完整，但共享 SaaS、整个 ASN 与易漂移 IP 风险最大 |
| blackmatrix7 Claude | — | 仅 `anthropic.com`、`claude.ai`、`cdn.usefathom.com` | 文件头标注 `UPDATED: 2025-06-06`；[固定源码](https://github.com/blackmatrix7/ios_rule_script/blob/8818705adee20571a856daf11c9fc69c4929109a/rule/Clash/Claude/Claude.list) | 缺少 `claude.com`、`claudeusercontent.com`、MCP 域名，不适合“尽量完整” |
| v2fly `openai` | 23 条：第一方主域、精确 Azure/Cloudflare/Imgix 主机、Advanced Voice 的 LiveKit、Web PubSub 正则和精确遥测主机 | — | 最近更新 [2026-07-13](https://github.com/v2fly/domain-list-community/commit/5dbad7124999abc285c74ecb4e80043657b39427)；[固定源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/openai) | 专用域名覆盖最完整；原始 v2fly 语法不能直接当 Clash classical 使用 |
| v2fly `anthropic` | — | 8 条：`anthropic.com`、`clau.de`、`claude.ai`、`claude.com`、两个 MCP 域、`claudeusercontent.com` 和一个精确 CDN 主机 | 最近更新 [2026-06-05](https://github.com/v2fly/domain-list-community/commit/1c383d5d90a01da183392d8e34dfc50b44cd7066)；[固定源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/anthropic) | 当前专用覆盖最完整，且没有泛 Cloudflare/Google/Azure 后缀 |
| MetaCubeX domain provider | 将 v2fly 分类转换成 Subconverter `clash-domain:` 可读取的 YAML；OpenAI 的正则项需单独兼容 | 同左 | `meta` 分支由自动流程持续生成；[固定 OpenAI](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/openai.yaml) / [固定 Anthropic](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/anthropic.yaml) | 最适合本项目直接引用，但它是 v2fly 的格式化下游，不算独立域名证据 |
| VPSDance | OpenAI 主域、Azure/LiveKit/遥测依赖、IPv4/IPv6、ASN 与 Web PubSub 正则 | Anthropic 主域、MCP、Auth0/Ghost/Cloudflare、遥测依赖、IPv4/IPv6 与 ASN | `main` 分支持续维护；[OpenAI](https://github.com/VPSDance/ai-proxy-rules/blob/main/rules/clash/openai.yaml) / [Anthropic](https://github.com/VPSDance/ai-proxy-rules/blob/main/rules/clash/anthropic.yaml) | Anthropic 表以 `clash-classic:` 引用；OpenAI 表因含 `DOMAIN-REGEX`，将有效增量归入本地 ext |
| cutethotw Claude | — | 合并多个社区来源，包含主域、第三方依赖、Google 静态资源、Anthropic IP/ASN、MCP 与关键词兜底 | [实时源码](https://github.com/cutethotw/ClashRule/blob/main/Rule/Claude.list) | 重复和共享流量很多，但符合覆盖优先策略 |
| SukkaW `ai.conf` | 6 个主域加 `DOMAIN-KEYWORD,openai`；主动注释掉 Statsig 和共享 Azure/Azure Blob 主机 | `anthropic.com`、`claude.ai`、`claude.com` | 最近更新 [2026-03-21](https://github.com/SukkaW/Surge/commit/4cff6af8996bef8a8a4d26cd5db86964d7f999b9)；[固定源码](https://github.com/SukkaW/Surge/blob/1dc1cc77725f064939bc8bdaa813b82c8e75b73a/Source/non_ip/ai.conf) | 精度优先，但 OpenAI Voice、新域名和 Claude MCP/内容域覆盖不如 v2fly |

## ACL4SSR 相对 MetaCubeX 的主要缺口

OpenAI 部分，ACL4SSR 当前没有显式覆盖 v2fly/MetaCubeX 的 `chat.com`、`chatgpt.site`、`crixet.com`、`oaistatsig.com`、`chatgpt.livekit.cloud`、`host.livekit.cloud`、`turn.livekit.cloud` 和 ChatGPT Web PubSub 动态主机等条目；部分带 `openai` 字符串的 Azure/CDN 主机只能依赖宽泛的 `DOMAIN-KEYWORD,openai` 命中。[ACL4SSR 固定源码](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)；[MetaCubeX OpenAI 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/openai.yaml)

Claude 部分，ACL4SSR 的显式后缀已经覆盖主站、API 和内容域；其关键词也会间接命中两个 Claude MCP 域和 Anthropic 专用 CDN。主要明确缺口是短域 `clau.de`。用 MetaCubeX `anthropic.yaml` 可以消除对关键词命中的依赖。[ACL4SSR 固定源码](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)；[MetaCubeX Anthropic 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/anthropic.yaml)

## 本地 OpenAI 扩展表

`rules/openai-ext.list` 当前有 40 条有效规则：

- 12 条域名规则：补 OpenAI 官方最新 allowlist 中 MetaCubeX / ACL4SSR 没有精确覆盖的 SendGrid、WorkOS、Cloudflare Challenge、Apple、Stripe、Sentry、Datadog 与 Statsig 条目。
- 23 条 ChatGPT Voice `/32`：与官方 `chatgpt-voice.json` 当前 `creationTime` 完全同步。
- 5 条 VPSDance 网络规则：`199.47.142.0/23`、两个 `/32`、`2604:f20::/32` 与 `IP-ASN,401518`。

Blackmatrix7 OpenAI 表另外带来 `IP-ASN,20473`、共享 SaaS 后缀和两个 `/32`。VPSDance OpenAI 表没有直接挂入 `config.ini`，因为它包含传统 Clash 不支持的 `DOMAIN-REGEX`；正则对应的 Web PubSub 主机已由 `DOMAIN-KEYWORD,chatgpt-async-webps-prod` 覆盖，其余有效增量已写入 ext。

## 共享基础设施风险与已接受的代价

“官方网络 allowlist”回答的是“要让产品正常工作，企业防火墙需要放行什么”，不等同于“这个域名上的所有流量都属于 OpenAI/Claude，应当全量走同一代理”。

- OpenAI 官方清单同时列出第一方域名和 `*.ct.sendgrid.net`、`*.intercom.io`、`*.intercomcdn.com`、`cdn.workos.com`、`challenges.cloudflare.com`、`humb.apple.com`、`js.stripe.com`、Sentry、Datadog 等共享第三方服务。[OpenAI 官方网络建议](https://help.openai.com/en/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps)
- 本仓库现在明确选择覆盖优先：除官方完整主机外，还接受社区表中的 `DOMAIN-SUFFIX,stripe.com`、`sentry.io`、`intercom.io`、Statsig 与关键词规则。它们会误匹配非 AI 租户，但不会造成漏分流。
- ACL4SSR 的 `DOMAIN-SUFFIX,auth0.com`、`intercom.io`、`intercomcdn.com`、`identrust.com` 以及 `DOMAIN-KEYWORD,openai/claude/anthropic` 会命中其他租户或无关域名；保留它是“完整性优先”的明确取舍。[ACL4SSR 固定源码](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)
- Blackmatrix7 OpenAI 的 `DOMAIN-SUFFIX,stripe.com`、`sentry.io`、`segment.io`、`auth0.com`、`intercom.io`，以及 `IP-ASN,20473` 会把大量非 OpenAI 流量送进 AI 策略；这是“宁可错杀”最明显的副作用，已明确接受。[blackmatrix7 OpenAI 实时源码](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/OpenAI/OpenAI.list)
- Anthropic 官方资料明确列出的 `api.anthropic.com`、`statsig.anthropic.com` 已被 `DOMAIN-SUFFIX,anthropic.com` 覆盖；社区表额外加入共享 Statsig 后缀，目的仅是覆盖未知或历史端点。[Anthropic 官方网络域名说明](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#approved-network-domains)

## 使用注意

- 运行配置使用 `meta`、`master` 分支 URL，以获得上游更新；本文证据链接固定到具体 commit，便于复核。
- MetaCubeX 两张专用表与 ACL4SSR 会有重复，但都指向 `🤖 AI 服务`，不会改变命中策略；代价是生成规则略多。
- 若将来选择“精度优先”，应按每个 AI 平台分别换成专用规则，而不是只删除 ACL4SSR，否则 Gemini、Copilot、Perplexity、xAI 等现有覆盖会丢失。
- OpenAI 官方要求 ChatGPT/Codex 的 WebSocket 连接到 `chatgpt.com`，并为 Voice 提供动态 IP 清单；域名部分由专用规则覆盖，`rules/openai-ext.list` 保存当前官方 Voice IP 快照。[OpenAI 官方网络建议](https://help.openai.com/en/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps)；[Voice IP JSON](https://openai.com/chatgpt-voice.json)
- Voice IP 清单会变化，验证时必须比较 `creationTime` 和所有 prefix；本次快照为 `2026-03-26T20:12:45.451356+00:00`。

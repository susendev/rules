# OpenAI / ChatGPT 与 Claude / Anthropic 分流规则研究

> 核对日期：2026-07-28。目标是为本项目的 Subconverter `ruleset=` 选择“覆盖较完整、不过度代理共享基础设施，并兼容传统 Clash 规则类型”的规则组合。

## 结论

推荐使用两层组合：

```ini
# OpenAI / ChatGPT：22 条域名规则 + 1 条动态 Web PubSub 兼容规则
ruleset=🤖 AI 服务,clash-domain:https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/openai.yaml
ruleset=🤖 AI 服务,[]DOMAIN-KEYWORD,chatgpt-async-webps-prod-

# Claude / Anthropic：8 条域名规则
ruleset=🤖 AI 服务,clash-domain:https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/anthropic.yaml

# 现有综合 AI 规则，继续覆盖 Gemini、Copilot、Perplexity、xAI 等平台
ruleset=🤖 AI 服务,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Ruleset/AI.list
```

理由：

- MetaCubeX 的 domain provider 输出来自 v2fly 分类，OpenAI 有 22 条标准域名、Anthropic 有 8 条，第一方域名和专用 CDN/Azure 主机覆盖明显比 ACL4SSR 单表更完整，同时避免对整个 Azure、Google、Cloudflare 后缀做代理。[OpenAI 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/openai.yaml)；[Anthropic 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/anthropic.yaml)；[MetaCubeX 构建流程](https://github.com/MetaCubeX/meta-rules-dat/blob/4178770badecb1b349fbcd62c737e0d7a2079729/.github/workflows/run.yml#L40-L50)
- v2fly 的第 23 条 OpenAI 规则是动态 Azure Web PubSub 正则；MetaCubeX classical 将其输出为 `DOMAIN-REGEX`，但传统 Clash 不支持该类型。本项目用唯一前缀 `chatgpt-async-webps-prod-` 的 `DOMAIN-KEYWORD` 等价补齐，避免把整个 `webpubsub.azure.com` 走 AI。[v2fly 固定源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/openai)；[MetaCubeX classical 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/classical/openai.list)
- `clash-domain:` 是 Subconverter 明确支持的规则集类型，会把 domain provider 转成标准 `DOMAIN` / `DOMAIN-SUFFIX` 规则，适合本项目当前的规则生成流程。[Subconverter 固定版本文档](https://github.com/tindy2013/subconverter/blob/a0d4eab28cb8b6c782d4ce5c3a918de4829b4a72/README-cn.md#L889-L906)；[类型定义](https://github.com/tindy2013/subconverter/blob/a0d4eab28cb8b6c782d4ce5c3a918de4829b4a72/src/handler/settings.cpp#L22-L23)
- ACL4SSR 仍有价值，因为它是一张 47 条的多平台 AI 聚合表；但其中的关键词和共享 SaaS 后缀不够精确，应作为完整性兜底，而不是 OpenAI/Claude 的唯一权威来源。[ACL4SSR 固定源码](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)
- 不建议再叠加 blackmatrix7 的 OpenAI/Claude 规则或 SukkaW 的整张 AI 规则：它们与上述组合大量重复，前者还有明显的扩大匹配风险，后者则刻意省略部分共享 Azure 主机以换取精度。[blackmatrix7 OpenAI](https://github.com/blackmatrix7/ios_rule_script/blob/8818705adee20571a856daf11c9fc69c4929109a/rule/Clash/OpenAI/OpenAI.list)；[blackmatrix7 Claude](https://github.com/blackmatrix7/ios_rule_script/blob/8818705adee20571a856daf11c9fc69c4929109a/rule/Clash/Claude/Claude.list)；[SukkaW AI](https://github.com/SukkaW/Surge/blob/1dc1cc77725f064939bc8bdaa813b82c8e75b73a/Source/non_ip/ai.conf)

## 来源对比

| 来源 | OpenAI / ChatGPT 覆盖 | Claude / Anthropic 覆盖 | 更新情况 | 判断 |
| --- | --- | --- | --- | --- |
| ACL4SSR `AI.list` | `openai.com`、`chatgpt.com`、`oaistatic.com`、`oaiusercontent.com`、`sora.com`，另有 `DOMAIN-KEYWORD,openai` | `anthropic.com`、`claude.ai`、`claude.com`、`claudeusercontent.com`，另有 `anthropic`/`claude` 关键词 | 文件最近一次提交为 [2026-01-25](https://github.com/ACL4SSR/ACL4SSR/commit/591b05e72d2d6c17d18a64e9af0c43e9b237b152) | 多 AI 平台聚合方便，但缺少部分新域名和高级语音主机，且关键词/共享后缀偏宽 |
| blackmatrix7 OpenAI | 35 条，包含第一方域名、LiveKit、Azure、监控/登录/支付第三方以及 IP/ASN | — | 文件头标注 `UPDATED: 2025-06-06`；[固定源码](https://github.com/blackmatrix7/ios_rule_script/blob/8818705adee20571a856daf11c9fc69c4929109a/rule/Clash/OpenAI/OpenAI.list) | 表面完整，但共享 SaaS、整个 ASN 与易漂移 IP 风险最大 |
| blackmatrix7 Claude | — | 仅 `anthropic.com`、`claude.ai`、`cdn.usefathom.com` | 文件头标注 `UPDATED: 2025-06-06`；[固定源码](https://github.com/blackmatrix7/ios_rule_script/blob/8818705adee20571a856daf11c9fc69c4929109a/rule/Clash/Claude/Claude.list) | 缺少 `claude.com`、`claudeusercontent.com`、MCP 域名，不适合“尽量完整” |
| v2fly `openai` | 23 条：第一方主域、精确 Azure/Cloudflare/Imgix 主机、Advanced Voice 的 LiveKit、Web PubSub 正则和精确遥测主机 | — | 最近更新 [2026-07-13](https://github.com/v2fly/domain-list-community/commit/5dbad7124999abc285c74ecb4e80043657b39427)；[固定源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/openai) | 专用域名覆盖最完整；原始 v2fly 语法不能直接当 Clash classical 使用 |
| v2fly `anthropic` | — | 8 条：`anthropic.com`、`clau.de`、`claude.ai`、`claude.com`、两个 MCP 域、`claudeusercontent.com` 和一个精确 CDN 主机 | 最近更新 [2026-06-05](https://github.com/v2fly/domain-list-community/commit/1c383d5d90a01da183392d8e34dfc50b44cd7066)；[固定源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/anthropic) | 当前专用覆盖最完整，且没有泛 Cloudflare/Google/Azure 后缀 |
| MetaCubeX domain provider | 将 v2fly 分类转换成 Subconverter `clash-domain:` 可读取的 YAML；OpenAI 的正则项需单独兼容 | 同左 | `meta` 分支由自动流程持续生成；[固定 OpenAI](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/openai.yaml) / [固定 Anthropic](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/anthropic.yaml) | 最适合本项目直接引用，但它是 v2fly 的格式化下游，不算独立域名证据 |
| SukkaW `ai.conf` | 6 个主域加 `DOMAIN-KEYWORD,openai`；主动注释掉 Statsig 和共享 Azure/Azure Blob 主机 | `anthropic.com`、`claude.ai`、`claude.com` | 最近更新 [2026-03-21](https://github.com/SukkaW/Surge/commit/4cff6af8996bef8a8a4d26cd5db86964d7f999b9)；[固定源码](https://github.com/SukkaW/Surge/blob/1dc1cc77725f064939bc8bdaa813b82c8e75b73a/Source/non_ip/ai.conf) | 精度优先，但 OpenAI Voice、新域名和 Claude MCP/内容域覆盖不如 v2fly |

## ACL4SSR 相对 MetaCubeX 的主要缺口

OpenAI 部分，ACL4SSR 当前没有显式覆盖 v2fly/MetaCubeX 的 `chat.com`、`chatgpt.site`、`crixet.com`、`oaistatsig.com`、`chatgpt.livekit.cloud`、`host.livekit.cloud`、`turn.livekit.cloud` 和 ChatGPT Web PubSub 动态主机等条目；部分带 `openai` 字符串的 Azure/CDN 主机只能依赖宽泛的 `DOMAIN-KEYWORD,openai` 命中。[ACL4SSR 固定源码](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)；[MetaCubeX OpenAI 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/openai.yaml)

Claude 部分，ACL4SSR 的显式后缀已经覆盖主站、API 和内容域；其关键词也会间接命中两个 Claude MCP 域和 Anthropic 专用 CDN。主要明确缺口是短域 `clau.de`。用 MetaCubeX `anthropic.yaml` 可以消除对关键词命中的依赖。[ACL4SSR 固定源码](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)；[MetaCubeX Anthropic 固定输出](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/anthropic.yaml)

## 共享基础设施风险

“官方网络 allowlist”回答的是“要让产品正常工作，企业防火墙需要放行什么”，不等同于“这个域名上的所有流量都属于 OpenAI/Claude，应当全量走同一代理”。

- OpenAI 官方清单同时列出第一方域名和 `*.ct.sendgrid.net`、`*.intercom.io`、`*.intercomcdn.com`、`cdn.workos.com`、`challenges.cloudflare.com`、`humb.apple.com`、`js.stripe.com`、Sentry、Datadog 等共享第三方服务。[OpenAI 官方网络建议](https://help.openai.com/en/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps)
- 因而不应为了“完整”添加 `DOMAIN-SUFFIX,cloudflare.com`、`DOMAIN-SUFFIX,azure.com`、`DOMAIN-SUFFIX,blob.core.windows.net`、`DOMAIN-SUFFIX,googleapis.com`、`DOMAIN-SUFFIX,stripe.com`、`DOMAIN-SUFFIX,sentry.io` 等规则。对确有必要的共享基础设施，只采用上游已验证的精确 `DOMAIN` 或受限正则。
- ACL4SSR 的 `DOMAIN-SUFFIX,auth0.com`、`intercom.io`、`intercomcdn.com`、`identrust.com` 以及 `DOMAIN-KEYWORD,openai/claude/anthropic` 会命中其他租户或无关域名；保留它是“完整性优先”的明确取舍。[ACL4SSR 固定源码](https://github.com/ACL4SSR/ACL4SSR/blob/4b461ca03b430c46d33b505cf4384e2f4f1be4b1/Clash/Ruleset/AI.list)
- blackmatrix7 OpenAI 的 `DOMAIN-SUFFIX,stripe.com`、`sentry.io`、`segment.io`、`auth0.com`、`intercom.io`，以及 `IP-ASN,20473` 会把大量非 OpenAI 流量送进 AI 策略；两个 `/32` 也可能随服务迁移而漂移，因此不建议本项目直接引用整表。[blackmatrix7 OpenAI 固定源码](https://github.com/blackmatrix7/ios_rule_script/blob/8818705adee20571a856daf11c9fc69c4929109a/rule/Clash/OpenAI/OpenAI.list)
- Anthropic 官方资料明确列出的 `api.anthropic.com`、`statsig.anthropic.com` 均已被 `DOMAIN-SUFFIX,anthropic.com` 覆盖，无需扩大到共享 Statsig 后缀。[Anthropic 官方网络域名说明](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#approved-network-domains)

## 使用注意

- 运行配置使用 `meta`、`master` 分支 URL，以获得上游更新；本文证据链接固定到具体 commit，便于复核。
- MetaCubeX 两张专用表与 ACL4SSR 会有重复，但都指向 `🤖 AI 服务`，不会改变命中策略；代价是生成规则略多。
- 若将来选择“精度优先”，应按每个 AI 平台分别换成专用规则，而不是只删除 ACL4SSR，否则 Gemini、Copilot、Perplexity、xAI 等现有覆盖会丢失。
- OpenAI 官方还要求 ChatGPT/Codex 的 WebSocket 连接到 `chatgpt.com`，并为 Voice 提供动态 IP 清单；域名部分已由专用规则覆盖，动态 Voice IP 不建议固化到本仓库。[OpenAI 官方网络建议](https://help.openai.com/en/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps)

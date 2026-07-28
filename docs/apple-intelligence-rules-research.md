# Apple Intelligence 分流规则研究

> 核对日期：2026-07-28。本文只记录已验证的一手或上游来源，不将泛 Apple 域名等同于 Apple Intelligence 域名。

## 结论

本项目采用 8 条规则，覆盖 Apple 官方列出的 Apple Intelligence、Siri、Search 主机，并补充主流社区规则中的 Apple Relay 与地区判定主机：

```text
DOMAIN,guzzoni.apple.com
DOMAIN-SUFFIX,smoot.apple.com
DOMAIN,apple-relay.cloudflare.com
DOMAIN,apple-relay.fastly-edge.com
DOMAIN,cp4.cloudflare.com
DOMAIN,apple-relay.apple.com
DOMAIN,apple-relay.mask.apple-dns.net
DOMAIN,gspe1-ssl.ls.apple.com
```

其中：

- `apple-relay.cloudflare.com`、`apple-relay.fastly-edge.com`、`cp4.cloudflare.com` 被 Apple 标为 Private Cloud Compute；`apple-relay.apple.com` 被标为 Apple Intelligence Extensions。[Apple 企业网络要求](https://support.apple.com/en-us/101555#apple-intelligence-siri-and-search)
- `guzzoni.apple.com` 是 Siri/听写，`*.smoot.apple.com` 是 Siri、Spotlight、Lookup、Safari、News、Messages、Music 等共用搜索服务。它们与 Apple Intelligence 相关，但不是 AI 专用主机。[Apple 企业网络要求](https://support.apple.com/en-us/101555#apple-intelligence-siri-and-search)
- `apple-relay.mask.apple-dns.net` 是社区维护的 ChatGPT Extension 补充项；Apple 当前企业网络页未列出，因此证据等级低于上述官方主机。[v2fly 固定版本源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/apple-intelligence)
- `gspe1-ssl.ls.apple.com` 是社区补充的地区判定主机。v2fly 明确提示它还可能影响中国 eSIM 地区判定、中国版地图、Apple Watch 健康通知和部分运营商 Wi-Fi Calling，因此代理它有跨服务副作用。[v2fly 固定版本源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/apple-intelligence)；[加入该主机的提交](https://github.com/v2fly/domain-list-community/commit/4fc40458c65b72e15ab7eae1136506977fd567b2)

## 上游规则对照

### v2fly/domain-list-community

当前 `data/apple-intelligence` 共 5 条：三个 `apple-relay` 主机、`apple-relay.mask.apple-dns.net` 和 `gspe1-ssl.ls.apple.com`。[固定版本源码](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/apple-intelligence)；[文件历史](https://github.com/v2fly/domain-list-community/commits/master/data/apple-intelligence)

### MetaCubeX/meta-rules-dat

其 Mihomo classical 输出同样是上述 5 条；构建流程每日检出 `v2fly/domain-list-community` 后生成规则，因此这里是下游产物，不算独立域名证据。[固定版本 classical 规则](https://github.com/MetaCubeX/meta-rules-dat/blob/42c2a789e19516f0a8bb2bca8eb1541505599940/geo/geosite/classical/apple-intelligence.list)；[构建流程](https://github.com/MetaCubeX/meta-rules-dat/blob/4178770badecb1b349fbcd62c737e0d7a2079729/.github/workflows/run.yml#L40-L50)

### SukkaW/Surge

独立 `apple_intelligence.conf` 包含 `apple-relay.apple.com`、`apple-relay.cloudflare.com`、`apple-relay.fastly-edge.com`、`cp4.cloudflare.com` 和 `gspe1-ssl.ls.apple.com`。它用 Apple 官方当前列出的 `cp4.cloudflare.com`，但没有 v2fly 的 `apple-relay.mask.apple-dns.net`。[固定版本源码](https://github.com/SukkaW/Surge/blob/08892b3a88a9107cba9b95b5daa3dd3b6766ded3/Source/non_ip/apple_intelligence.conf)；[创建独立规则的提交](https://github.com/SukkaW/Surge/commit/08892b3a88a9107cba9b95b5daa3dd3b6766ded3)

## 明确排除

以下内容不纳入 Apple Intelligence 规则：

- Apple ID：`account.apple.com`、`appleid.cdn-apple.com`、`idmsa.apple.com`、`gsa.apple.com` 属于账号认证。[Apple Account 源说明](https://support.apple.com/en-us/101555#apple-account)
- iCloud Private Relay：`mask.icloud.com`、`mask-h2.icloud.com`、`mask-api.icloud.com` 属于 iCloud Private Relay。[Apple iCloud 源说明](https://support.apple.com/en-us/101555#icloud)；[v2fly 独立分类](https://github.com/v2fly/domain-list-community/blob/4c822d86d3ab2a39db795ecf25c975e700dcdb07/data/icloudprivaterelay)
- CloudKit 与 `gateway.icloud.com` 属于 iCloud/CloudKit 内容和系统资产，不是 Apple Intelligence 专用流量。[Apple iCloud 与内容源说明](https://support.apple.com/en-us/101555#icloud)
- `app-site-association.cdn-apple.com`、`app-site-association.networking.apple` 属于 Universal Links、Handoff、App Clips、SSO 使用的 Associated Domains。[Apple Associated Domains 源说明](https://support.apple.com/en-us/101555#associated-domains)

这 8 条适合“Apple Intelligence 连同 Siri/Search 一并走 AI 策略”的目标；若追求严格的 AI 专用最小集，应另外评估是否移除共享的 `guzzoni`、`smoot` 与 `gspe`。

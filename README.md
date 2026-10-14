# openclash-config

OpenClash 自定义订阅转换模板（分流规则与节点解耦）。

## 文件

| 文件 | 规则源 | 是否需要代理 |
| --- | --- | --- |
| `my_fallback.ini` | `raw.githubusercontent.com` | **需要**（与原方案作者写法一致） |
| `my_fallback_jsdelivr.ini` | `testingcf.jsdelivr.net` | 不需要（CDN 直连） |

两份**功能完全等价**，只是规则源域名不同。分流设计：

- 国内走直连（ACL4SSR 的 ChinaDomain / ChinaCompanyIp / LocalAreaNetwork + `GEOIP,CN,no-resolve`）
- **其余全部走代理**（兜底 `[]FINAL`）
- 策略组 4 个：`🚀 节点选择`(select) / `♻️ 自动容灾`(fallback) / `⚡ 自动测速`(url-test) / `🎯 全球直连`(select)
- 容灾顺位：**美国 → 日本 → 新加坡 → 台湾 → 韩国 → 英国 → 土耳其 → 香港 → 其余**

## 用于 OpenClash

【服务 → OpenClash → 配置订阅】新增条目：

| 界面项 | 值 |
| --- | --- |
| 订阅地址 | 你的机场订阅链接（原样粘贴） |
| 在线订阅转换 | 勾选 |
| 转换服务地址 | `http://<转换器IP>:25500/sub`（末尾**不带问号**） |
| 模板名称 | Custom Template（自定义模板） |
| 自定义模板地址 | 见下方二选一 |

`my_fallback.ini`（主用，raw.github）：

```
https://raw.githubusercontent.com/xhmTmax530/openclash-config/main/my_fallback.ini
```

`my_fallback_jsdelivr.ini`（备用，CDN）：

```
https://testingcf.jsdelivr.net/gh/xhmTmax530/openclash-config@main/my_fallback_jsdelivr.ini
```

## 重要前提

`my_fallback.ini` 里的 `raw.githubusercontent.com` 在国内部分线路会被 **SNI 阻断**
（TCP 能连、TLS Client Hello 被 RST）。若转换器直连该域名，subconverter 会
**静默丢弃全部规则**，只留一条兜底 `MATCH` —— 表面上仍返回 HTTP 200。

因此使用 `my_fallback.ini` 前，必须让 subconverter 走代理：

```toml
# subconverter 的 pref.toml
proxy_ruleset = "http://<用户>:<密码>@<代理IP>:<端口>"
```

若代理不可用，改用 `my_fallback_jsdelivr.ini`（其规则源在国内可直连，无需代理）。

## 注意

- 本仓库不含任何订阅地址、节点信息或凭据。
- 模板内的规则地址若含 `testingcf.jsdelivr.net`，注意其有 12 小时 CDN 缓存。

# openclash-config

OpenClash 自定义订阅转换模板（分流规则与节点解耦）。

## 文件

| 文件 | 规则源 | 转换器需要代理 |
| --- | --- | --- |
| `my_fallback.ini` | `raw.githubusercontent.com` | **需要**（与原方案作者写法一致） |
| `my_fallback_jsdelivr.ini` | `testingcf.jsdelivr.net` | 不需要（CDN 直连，应急备用） |

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
| 自定义模板地址 | 见下 |

主用（raw.github，与原方案作者一致）：

```
https://raw.githubusercontent.com/xhmTmax530/openclash-config/main/my_fallback.ini
```

应急备用（CDN，无需代理）：

```
https://testingcf.jsdelivr.net/gh/xhmTmax530/openclash-config@main/my_fallback_jsdelivr.ini
```

## 重要前提：转换器必须配置代理

`raw.githubusercontent.com` 在国内部分线路会被 **SNI 阻断**（TCP 能连、TLS Client Hello 被 RST）。

若转换器直连该域名，subconverter 会 **静默丢弃** 内容，且仍返回 HTTP 200：

- 模板拉不到 → 换成 subconverter **自带的默认模板**（几十个策略组，一万多条规则）
- 规则拉不到 → 规则被丢弃，只剩一条 `MATCH`

**subconverter 有三个独立的代理开关，必须分别设置**（`pref.toml`）：

```toml
proxy_config       = "http://<用户>:<密码>@<代理IP>:<端口>"   # 下载「模板文件」
proxy_ruleset      = "http://<用户>:<密码>@<代理IP>:<端口>"   # 下载「规则集 .list」
proxy_subscription = "NONE"                                   # 下载「机场订阅」（通常直连即可）
```

> 只设 `proxy_ruleset` 是不够的 —— 模板文件由 `proxy_config` 控制，两者不通用。
> 漏设 `proxy_config` 时不会有任何报错，只会静默换成内置默认模板。

若代理不可用，改用 `my_fallback_jsdelivr.ini`（其规则源在国内可直连，无需代理）。

## 注意

- 本仓库不含任何订阅地址、节点信息或凭据。
- 若规则源使用 `testingcf.jsdelivr.net`，注意其有 12 小时 CDN 缓存。

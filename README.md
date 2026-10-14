# openclash-config

OpenClash 自定义订阅转换模板（分流规则与节点解耦）。

## 文件

- `my_fallback.ini` — subconverter 外部配置模板。国内直连、其余全部走代理、节点按国家顺位 fallback 容灾。

## 用于 OpenClash

【服务 → OpenClash → 配置订阅】新增条目：

| 界面项 | 值 |
| --- | --- |
| 订阅地址 | 你的机场订阅链接（原样粘贴） |
| 在线订阅转换 | 勾选 |
| 转换服务地址 | `http://<N1的IP>:25500/sub` |
| 模板名称 | Custom Template（自定义模板） |
| 自定义模板地址 | 见下方二选一 |

两种引用方式（二选一）：

- GitHub CDN（换设备 / 重装后可直接用，推荐）：
  `https://testingcf.jsdelivr.net/gh/xhmTmax530/openclash-config@main/my_fallback.ini`
- 本地（改完立即生效，无 CDN 缓存）：
  `mytpl/my_fallback.ini`

## 注意

- 模板内的规则地址一律使用 `testingcf.jsdelivr.net`，不能写 `raw.githubusercontent.com` ——
  转换器所在的设备无法解析该域名，会导致规则被静默丢弃。
- 本仓库不含任何订阅地址、节点信息或凭据。

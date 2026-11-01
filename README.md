# proxy-configs

常用代理软件的配置模板。**全部以 Loon 配置为基准**：四个模板的策略组命名、地区筛选正则、分流规则顺序完全同构，换软件不用重新理解一套结构。节点、机场订阅、MITM 证书私钥均已替换为占位符，可直接公开。

## 基准与对应关系

以 `loon/loon-template.conf` 为蓝本，其他三个模板按同一张映射表对齐：

| Loon 结构 | Surge | Quantumult X | Clash (mihomo) |
| --- | --- | --- | --- |
| `[Proxy Group]` 场景组（AI平台/电报消息/国外媒体…） | `[Proxy Group]` 同名组 | `[policy]` 同名 static 组 | `proxy-groups` 同名组 |
| `[Remote Filter]` 六地区正则 | 地区组 `policy-regex-filter` | `server-tag-regex` | 地区组 `filter`（url-test） |
| `[Remote Rule]` 分流顺序 | `[Rule]` RULE-SET 同序 | `[filter_remote]` 同序 | `rule-providers` + `rules` 同序 |
| FINAL / 节点选择 / 全球代理 | 同名 | 🚀 节点选择 / 🌍 全球代理 | 同名 |

分流顺序（四个模板一致）：**广告拦截 → 国内AI → 海外AI分品牌（OpenAI / Claude / Gemini / Copilot / JetBrains / AI平台） → TestFlight / 苹果 / 私有种子(PT) / 电报 → 国内垂直流媒体 → ChinaMedia → YouTube / Netflix / 巴哈 → Google → GlobalMedia → 微软服务直连（Microsoft.list）→ ChinaMaxNoMedia 直连 → Global 走代理**。

2026-11-01 策略组解耦优化：
- **规则与策略组 1:1 独立解耦**：每个规则/规则集均拥有专属策略组，杜绝规则硬编码与大池混跑。
  - **广告拦截**：广告与劫持规则指向独立的「广告拦截」策略组（默认 REJECT，遇误杀可一键切 DIRECT 应急）。
  - **AI 细分独立策略组**：国内 AI 指向「国内AI」（默认 DIRECT 可切节点）；海外 AI 全面细分为「OpenAI」、「Claude」、「Gemini」、「Copilot」、「JetBrains」与「AI平台」（兜底其它海外 AI），便于针对封锁 IP 严格的平台（如 Claude/OpenAI）单独指定干净节点。
  - **私有种子 (PT)**：从大内网/国内直连中剥离出「私有种子」专属策略组（默认 DIRECT，可指定下载节点或直连防死）。

2026-10-30 分流审计（4 模板 × 逐域名命中探测 + 对抗验证）后加入的修正：

- **微软服务直连**：ChinaMaxNoMedia 大表缺 `microsoft.com` 主域，微软全家桶（bing/outlook/azure/live/msn…）实际漏到全球代理；四模板统一新增 `Microsoft.list`（"微软服务"组，默认 DIRECT 可切节点）。
- **裸后缀豁免**：ChinaMaxNoMedia 携带裸 `DOMAIN-SUFFIX,ms` / `cn`，截杀 `aka.ms`/`1drv.ms` 等 22 个微软基础设施域与 futu.cn 等 9 个境外券商 .cn 域；统一前置豁免。
- **广告表误杀豁免（Clash）**：上游 Advertising_Domain.yaml 误收网易云/腾讯视频/爱奇艺接口、4paradigm、gvt2/ggpht/cookielaw/onetrust，9 条前置豁免。
- **QX 局域网顶置**：AdvertisingTest 表内含 `10.72.25.0/24` REJECT 会击穿内网直连，Lan.list 顶置为第一优先。
- **QX GEOIP 补齐**：补 KR/TW/CN geoip 兜底，与其他三模板一致。
- **Loon 死源修复**：kelee.one 插件源→GitHub 镜像 mihoyo-typ/KeleeOne；Sub-Store 解析器 GitLab 旧址→GitHub releases；Tartarus Loon-Script 五条 404 脚本置为 disabled。

规则统一来自两个仓库（四个模板引用同一批文件，行为一致）：
- [shjliu911/ios-rule-script-mirror](https://github.com/shjliu911/ios-rule-script-mirror) — blackmatrix7 规则镜像（按软件子目录取用）
- [shjliu911/surge-ai-lists](https://github.com/shjliu911/surge-ai-lists) — AI 分流（Surge/.list、Clash/.yaml、QX quanx/ 三种格式）

## 目录

| 目录 | 软件 | 说明 |
| --- | --- | --- |
| `loon/` | Loon (iOS) | **基准模板**。KOP 资源解析器 + `[Remote Filter]` 地区正则 + 场景化策略组（Qure 图标） |
| `surge/` | Surge for Mac / iOS | 国内 DoH + 游戏主机 real-ip 白名单；地区组用 `policy-regex-filter` 挂机场订阅 |
| `quantumultx/` | Quantumult X (iOS) | KOP-XIAO 解析器 + ip-api 中文节点信息；`server-tag-regex` 地区组 |
| `clash/` | Clash / OpenClash (mihomo) | `proxy-providers` 挂订阅 + `rule-providers` 远程规则；地区组 url-test 自动选优 |

## 使用

各模板顶部注释有对应软件的操作步骤。通用占位符：

- `YOUR-AIRPORT-SUBSCRIPTION-URL` — 机场订阅地址
- `YOUR_CA_PASSPHRASE` / `YOUR_CA_P12_BASE64` — MITM 证书（在各软件 UI 里重新生成即可，无需手填）
- `YOUR_SS_PASSWORD` 等 — 本地示例节点的密码

## 免责声明

模板仅用于个人学习与备份。请遵守当地法律法规。

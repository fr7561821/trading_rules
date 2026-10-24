# trading-rules

交易客户端 / 交易网站分流规则集，供代理客户端远程引用。所有规则统一走 **「交易」** 策略组。

## 文件说明

| 文件 | 适用客户端 | 格式 |
| --- | --- | --- |
| `trading.yaml` | Clash / Mihomo / Clash Verge | YAML，含 `payload:` 根键，每行 `TYPE,VALUE` |
| `trading.list` | Surge / Loon / Shadowrocket | 纯文本，一行一条 `TYPE,VALUE` |

> 旧文件 `rules.yaml` 采用「`- ` 前缀 + 行尾带策略名」的写法，两种客户端都可能解析失败，已弃用。

## 覆盖范围

**进程规则（PROCESS-NAME）**

- ATAS：`OFT.Platform.exe`（经典版 / X）、`OFT.Platform.Updater.exe`（更新程序）
- NinjaTrader、Sierra Chart、Rithmic Trader Pro、Bookmap
- DayTrader 32/64 位（`daytrader64.exe`、`daytraderX64.exe`）
- TradingView 桌面版（`tradingview.exe`）

**域名关键字（DOMAIN-KEYWORD）**

- `tradingview`、`topstep`

## 引用方式

### Clash / Mihomo

```yaml
rule-providers:
  trading:
    type: http
    behavior: classical
    url: "https://raw.githubusercontent.com/fr7561821/trading_rules/main/trading.yaml"
    path: ./ruleset/trading.yaml
    interval: 86400

rules:
  - RULE-SET,trading,交易
```

### Surge

```ini
[Rule]
RULE-SET,https://raw.githubusercontent.com/fr7561821/trading_rules/main/trading.list,交易
```

## 注意事项

- **规则集文件里不要写策略名。** Surge 官方手册明确要求外部规则集文件
  "written without the policy component"；Clash / Mihomo 的 `classical`
  行为同样只识别 `TYPE,VALUE` 两段。若写成
  `PROCESS-NAME,OFT.Platform.exe,交易`，客户端会把
  `OFT.Platform.exe,交易` 整体当作进程名去匹配，规则将永远不生效，
  且只在日志里留一条警告——交易流量会静默走错通道。
- **`PROCESS-NAME` 需要客户端开启进程查找。** Mihomo / Clash 通过
  `find-process-mode`（`always` / `strict` / `off`，默认 `strict`）控制，
  未开启时进程类规则不生效；建议设为 `always`，或至少确认你的客户端
  支持 Windows 进程名匹配。
- **规则顺序**：自上而下匹配，把交易规则放在 `GEOIP` / `MATCH` 等兜底规则之前。
- **缓存**：`raw.githubusercontent.com` 有缓存，更新文件后客户端可能延迟若干分钟
  才拉到新内容，可用 `update-interval` 控制刷新频率。

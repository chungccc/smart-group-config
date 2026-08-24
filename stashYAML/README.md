# Stash Smart Group Config

一套以 **地区节点自动分类 + URL-Test 智能优选 + GEOSITE 规则分流** 为核心的 Stash 配置方案。

本项目通过 **Proxy Provider + 正则 Filter** 自动筛选不同地区的节点，并使用 `url-test` 策略组持续测试节点延迟，在香港、日本、新加坡、美国等地区节点之间自动选择更合适的节点。

> **核心理念：**
>
> **机场订阅 → 节点过滤 → 地区分类 → URL-Test 自动优选 → 规则分流**

适合希望在 Stash 中使用多个机场节点，同时减少手动管理节点和策略组的用户。

---

## ✨ 核心特性

* 🌏 **地区自动分类**：自动识别香港、日本、新加坡、美国节点
* ⚡ **URL-Test 自动优选**：定时测试节点并自动选择低延迟节点
* 📡 **机场订阅管理**：通过 `proxy-providers` 统一管理机场订阅
* 🚫 **无效节点过滤**：自动排除机场公告、流量信息等非节点内容
* 🎯 **独立业务策略组**：TikTok、OpenAI、Emby 等服务独立分流
* 📚 **GEOSITE 规则**：使用内置 GEOSITE 数据进行域名分类
* 🛡️ **广告拦截**：支持广告规则集拦截
* 🚀 **QUIC 拦截**：避免部分流量绕过代理策略
* 🌐 **国内外自动分流**：LAN / CN 默认直连，其余流量进入代理策略

---

# ⚙️ 配置结构

整个配置的工作流程如下：

```text
机场订阅
    ↓
Proxy Provider
    ↓
无效节点过滤
    ↓
地区 Filter
    ↓
HKG / SGP / JPN / USA
    ↓
URL-Test 自动测速
    ↓
业务策略组
    ↓
规则分流
```

新增机场节点后，只要节点名称包含对应地区关键词，即可自动进入相应的地区测速组。

---

# 📡 机场订阅

本配置使用 `proxy-providers` 管理机场订阅。

示例：

```yaml
proxy-providers:
  airportA:
    filter: "^(?!.*(流量|到期|套餐|官网|刷新|订阅|更新|失联|邮箱|IPV6|官方)).*$"

    interval: 10800

    url: # 删除本注释后填写订阅链接
```

其中：

* `airportA`：机场订阅名称
* `url`：填写机场订阅地址
* `interval: 10800`：订阅定期更新
* `filter`：过滤机场订阅中的无效内容

---

## 🚫 无效节点过滤

部分机场订阅会将以下内容混入节点列表：

```text
流量
到期
套餐
官网
刷新
订阅
更新
失联
邮箱
IPV6
官方
```

这些通常不是实际代理节点，因此配置使用：

```yaml
filter: "^(?!.*(流量|到期|套餐|官网|刷新|订阅|更新|失联|邮箱|IPV6|官方)).*$"
```

自动排除包含这些关键词的内容。

如果你的机场存在其他公告或无效节点名称，也可以自行继续添加关键词。

例如：

```yaml
filter: "^(?!.*(流量|到期|套餐|官网|刷新|订阅|更新|失联|邮箱|IPV6|官方|测试节点)).*$"
```

---

# 🌏 地区自动分类

本配置通过节点名称中的地区关键词自动分类。

目前包含：

* 🇭🇰 香港 `HKG`
* 🇸🇬 新加坡 `SGP`
* 🇯🇵 日本 `JPN`
* 🇺🇸 美国 `USA`

---

## 🇭🇰 HKG

自动匹配包含以下关键词的节点：

```text
香港
香港🇭🇰
🇭🇰
HK
HKG
Hong Kong
```

示例：

```yaml
- name: HKG
  filter: ".*(香港|香港🇭🇰|🇭🇰|HK|HKG|Hong Kong).*"
  type: url-test
  interval: 120
  tolerance: 50
  lazy: true
  use:
    - airportA
```

---

## 🇸🇬 SGP

自动匹配：

```text
新加坡
新加坡🇸🇬
🇸🇬
SG
SGP
Singapore
```

符合条件的节点会自动进入：

```text
SGP
```

测速组。

---

## 🇯🇵 JPN

自动匹配：

```text
日本
日本🇯🇵
🇯🇵
JP
JPN
Japan
```

符合条件的节点自动进入：

```text
JPN
```

测速组。

---

## 🇺🇸 USA

自动匹配：

```text
美国
美国🇺🇸
🇺🇸
US
USA
United States
United States of America
```

符合条件的节点自动进入：

```text
USA
```

测速组。

---

# ⚡ URL-Test 自动优选

每个地区使用独立的 `url-test` 策略组。

例如：

```text
机场节点
    ↓
地区 Filter
    ↓
HKG
    ↓
自动测速
    ↓
选择当前更合适的节点
```

用户无需手动选择具体节点。

例如香港节点：

```text
HK 01
HK 02
HK 03
Hong Kong 01
香港 02
```

只要名称符合 Filter 条件，就会自动进入：

```text
HKG
```

然后由 `url-test` 自动进行测速和选择。

---

# 🧭 策略组结构

## 🌐 General

默认代理策略组。

General 由多个地区测速组组成：

```text
General
 ├── HKG
 ├── SGP
 └── JPN
```

大部分需要代理的流量最终都会进入 General。

你可以根据自己的网络环境，在不同地区之间进行选择。

---

## 🎵 TikTok

TikTok 使用独立策略组：

```text
TikTok
 ├── SGP
 └── JPN
```

方便针对 TikTok 单独选择地区出口。

---

## 🤖 OpenAI

OpenAI 使用独立策略组：

```text
OpenAI
 ├── SGP
 ├── JPN
 └── DIRECT
```

相关流量通过 GEOSITE 规则自动进入 OpenAI 策略组。

---

## 🎬 Emby

Emby 使用独立策略组：

```text
Emby
 ├── SGP
 ├── JPN
 └── DIRECT
```

配置中同时包含指定 Emby 域名规则。

---

## 🚫 广告拦截

广告策略组提供：

```text
广告拦截
 ├── DIRECT
 └── REJECT
```

广告规则通过远程 Rule Provider 获取。

主要用于：

* 广告域名拦截
* 第三方广告过滤规则

---

# 📜 规则分流

本配置主要使用：

```text
GEOSITE
+
DOMAIN-SUFFIX
+
PROCESS-NAME
+
IP-ASN
+
GEOIP
```

进行流量分类。

---

## 常见服务

以下服务会通过 GEOSITE 自动进入对应策略：

```text
Google
GitHub
Netflix
Disney+
Spotify
YouTube
Telegram
Twitter
Instagram
Facebook
TikTok
Steam
Microsoft
OpenAI
```

例如：

```yaml
- GEOSITE,netflix,General
- GEOSITE,youtube,General
- GEOSITE,telegram,General
- GEOSITE,tiktok,tiktok
- GEOSITE,openai,openai
```

---

# 🇨🇳 国内流量

局域网和中国大陆 IP 默认直连：

```yaml
- GEOIP,LAN,DIRECT
- GEOIP,CN,DIRECT
```

部分中国大陆服务也使用 GEOSITE 规则进行直连，例如：

```yaml
- GEOSITE,steam@cn,DIRECT
- GEOSITE,microsoft@cn,DIRECT
```

---

# 🚀 QUIC 处理

配置中包含 QUIC 拦截规则：

```yaml
- SCRIPT,quic,REJECT
```

对应脚本：

```yaml
script:
  shortcuts:
    quic: network == 'udp' and dst_port == 443
```

用于阻止 UDP 443 的 QUIC 流量。

这样可以避免部分应用绕过正常的代理规则。

---

# 🌐 DNS

配置提供国内 DNS 服务器：

```yaml
default-nameserver:
  - 114.114.115.115
  - 119.28.28.28
  - 223.6.6.6
  - system
```

同时使用 DoH：

```yaml
nameserver:
  - https://223.5.5.5/dns-query
  - https://223.6.6.6/dns-query
  - https://1.12.12.12/dns-query
  - https://120.53.53.53/dns-query
```

并启用：

```yaml
follow-rule: null
```

使 DNS 查询尽可能配合规则进行处理。

---

# ➕ 添加新的机场

如果需要增加新的机场订阅，可以在：

```yaml
proxy-providers:
```

下新增一个 Provider。

例如：

```yaml
proxy-providers:

  airportA:
    filter: "^(?!.*(流量|到期|套餐|官网|刷新|订阅|更新|失联|邮箱|IPV6|官方)).*$"
    interval: 10800
    url: 你的机场订阅链接

  airportB:
    filter: "^(?!.*(流量|到期|套餐|官网|刷新|订阅|更新|失联|邮箱|IPV6|官方)).*$"
    interval: 10800
    url: 你的机场订阅链接
```

然后在地区策略组的 `use` 中加入：

```yaml
use:
  - airportA
  - airportB
```

例如：

```yaml
- name: HKG
  type: url-test
  filter: ".*(香港|香港🇭🇰|🇭🇰|HK|HKG|Hong Kong).*"
  interval: 120
  tolerance: 50
  lazy: true
  use:
    - airportA
    - airportB
```

这样两个机场中的香港节点都会自动进入 HKG 测速池。

---

# 🔄 多机场工作方式

例如你同时使用：

```text
机场A
机场B
```

两个机场。

配置结构可以是：

```text
机场A ─┐
       ├── HKG ──┐
机场B ─┘         │
                 ├── General
机场A ─┐         │
       ├── JPN ──┤
机场B ─┘         │
                 ↓
              网络流量
```

多个机场的同地区节点会自动汇集到对应地区策略组。

无需手动把每一个节点添加到策略组中。

---

# ⚠️ 使用前注意

## 1. 必须填写机场订阅地址

找到：

```yaml
url:
```

填写自己的订阅链接。

---

## 2. 修改机场名称

默认示例：

```yaml
airportA:
```

可以根据自己的机场修改：

```yaml
机场A:
机场B:
```

但修改后需要同步调整所有策略组中的：

```yaml
use:
```

例如：

```yaml
use:
  - 机场A
  - 机场B
```

---

## 3. 节点名称必须包含地区信息

地区分组依赖节点名称进行 Filter。

例如香港节点建议包含：

```text
HK
香港
HKG
Hong Kong
🇭🇰
```

否则可能无法被 HKG 策略组识别。

---

# 📌 推荐使用方式

整个配置推荐按照以下方式理解：

```text
机场负责提供节点
        ↓
Proxy Provider 负责管理订阅
        ↓
Filter 负责过滤无效节点
        ↓
地区策略组负责分类
        ↓
URL-Test 负责自动测速
        ↓
业务策略组负责服务选择
        ↓
Rules 负责自动分流
```

---

# 🚀 总结

这份配置的重点并不是维护大量手动节点，而是通过：

```text
Proxy Provider
+
Filter
+
Regional URL-Test
+
GEOSITE Rules
```

建立自动化的节点管理和流量分流体系。

只需要：

1. 添加机场订阅
2. 在 `use` 中加入机场
3. 保证节点名称包含地区信息

节点就可以自动进入：

```text
HKG
SGP
JPN
USA
```

等地区测速组。

随后由 `url-test` 自动选择更合适的节点，再通过规则系统完成不同服务的自动分流。

> **推荐理念：**
>
> **机场负责提供节点，Filter 负责筛选，地区组负责分类，URL-Test 负责选择，Rules 负责分流。**


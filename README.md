# 策略组与分流配置说明

本配置采用多机场、多地区、多质量等级的节点体系，结合 Smart Group 智能节点选择，实现高效、稳定、精细化的流量管理。

> **Personal Configuration Notice**
> This configuration is designed exclusively for personal use. Node grouping, quality classification, rule routing, and Smart Group strategies are customized according to the author's actual network environment and usage requirements. It is **not intended to be used as a general-purpose configuration or template**.

---

## 一、节点前缀 / 后缀体系

每个机场的 Provider 都设定了统一的前缀，用于区分不同机场来源，并为节点添加后缀 `PRM / STD / BSC` 用于质量划分。

### PRM — Premium

高质量节点。

* 用于 Instagram、Telegram、TikTok、ChatGPT 等对实时性与速度要求较高的应用。
* 香港、新加坡、日本均提供 PRM 节点。

### STD — Standard

标准质量节点。

* 作为 PRM 节点的备用。
* 适用于日常网页浏览及一般网络使用。

### BSC — Basic

基础质量节点。

* 多为低成本或直连类机场节点。
* 适用于 Steam 下载、Emby、PT 以及其他大流量使用场景。

节点后缀同时也是后续策略组筛选节点的重要依据。

香港、新加坡、日本等常用地区均按照 `PRM / STD / BSC` 进行三级质量分组，以便根据实际需求选择合适的节点等级。

---

## 二、地区化分组

常用地区包括：

* 香港
* 新加坡
* 日本

这些地区均提供 `PRM / STD / BSC` 三级质量分组。

美国、韩国、台湾等不常用地区则采用独立策略组，根据实际需求使用。

同时，各机场也维护其专属策略组，包含该机场对应的香港、新加坡等地区节点，方便按照机场来源进行选择。

---

## 三、Smart 智能策略

在 `PRM / STD / BSC` 分类下，多数策略组使用 Smart Group 模式。

Smart Group 主要用于：

* 自动进行节点延迟测试
* 评估节点可用性
* 综合判断带宽与稳定性
* 根据节点质量自动选择合适节点
* 减少人工选择节点的需要

Smart Group 构成整个配置中的主要节点选择逻辑，可覆盖绝大多数日常使用场景。

---

## 四、内容 / 服务专用策略组

### YouTube-AdFree

独立的 YouTube 去广告线路策略组。

可根据实际需求选择独立节点或智能节点组。

### Emby

热门 Emby 服务域名使用独立规则进行匹配，并交由专用策略组处理。

这样可以避免 Emby 流量被错误分配到兜底策略或高质量节点。

### Test

用于 IP、ASN、地理位置以及网络环境检测。

目前主要包含：

* IP 检测
* ASN 检测
* GeoIP 检测
* 网络线路检测

后续可以根据实际使用情况继续增加检测域名。

---

## 五、安全与广告过滤

配置中部署了基于规则集的广告拦截策略，包括：

* AdBlock 基础规则
* AdBlock Plus 扩展规则

根据规则优先级自动进行匹配。

如果 OpenWrt 中同时启用了 AdBlock 插件，则优先使用 AdBlock 进行广告拦截，OpenClash 中的广告拦截策略主要用于放行及补充处理。

---

## 六、FavSites 常用网站

常用访问的网站统一整理至 `FavSites` 规则集。

这些网站根据实际使用情况进行维护，并使用独立的 `FavSites` 策略组进行分流。

主要目的：

* 避免常用网站落入兜底策略
* 避免错误节点分配
* 提高常用网站访问稳定性
* 方便集中维护个人常用网站规则

`FavSites` 并不是通用的网站代理规则集，而是根据个人实际使用情况建立的规则集合。

---

## 七、直连网站 / 特殊域名处理

针对容易被误判走代理的国内系统及特殊服务，建立独立的直连规则。

主要包括：

* 报名系统
* 管理系统
* 支付相关服务
* 部分订阅服务
* 其他需要强制直连的域名

这些规则用于避免特殊域名受到默认分流逻辑影响，确保相关服务能够正常访问。

---

# 2026 年 8 月 24 日更新

## 九、规则集集中维护

将个人使用的分流规则逐步从主配置中独立出来，并统一整理至 GitHub `Ruleset` 仓库进行维护。

通过独立 Rule Provider 管理不同类型的规则，使主配置与具体域名规则解耦。

后续可以直接在 GitHub 中维护规则，而无需频繁修改主配置文件。

---

## 十、FavSites 规则集

新增 `FavSites.yaml`，用于集中维护个人日常使用以及需要代理访问的网站。

该规则集根据实际访问记录持续补充，并作为个人常用网站的专用分流规则。

主要用于避免相关域名因为未命中特定规则而进入兜底策略。

`FavSites` 仅代表个人实际使用环境中的代理需求，不作为通用代理规则集。

---

## 十一、Direct 直连规则集

新增 `Direct.yaml`，用于集中管理需要强制直连的特殊域名。

主要包括：

* 报名系统
* 管理系统
* 订阅服务
* 其他容易因默认规则或地区判断而被错误代理的域名

通过独立规则集统一指定直连策略，可以减少主配置中的重复规则，并方便后续维护。

---

## 十二、IP / ASN / Geo 检测专用策略

新增 `IP-Test.yaml`，并建立独立的 `Test` 策略组。

目前主要用于：

* IP 地址检测
* ASN 检测
* 地理位置检测
* 网络线路检测

目前使用的检测域名包括：

```text
ip.sb
ipwho.is
ipapi.is
ip.skk.moe
iplark.com
bgp.tools
open.cachefly.net
```

通过独立的 `Test` 策略组，可以在测试不同节点时快速确认实际出口 IP、ASN、地区以及网络线路。

---

## 十三、规则集自动更新

GitHub 中维护的 Rule Provider 统一设置为每 `3600` 秒检查一次更新。

```yaml
interval: 3600
```

即每小时自动检查 GitHub 上的规则是否发生变化。

后续新增或修改规则时，无需频繁修改主配置即可同步更新。

---

## 十四、规则体系进一步细化

根据近期实际使用情况，对未命中特定规则的域名进行持续整理。

将实际访问过程中发现的以下类型域名逐步加入对应规则集：

* 海外服务
* CDN
* 下载服务
* 媒体服务
* 开发服务
* AI 服务
* 图片服务
* 其他个人常用网站

同时继续区分代理、直连以及测试类流量，使规则匹配更加明确。

---

## 十五、节点筛选体系进一步细化

延续原有 `PRM / STD / BSC` 质量等级体系，并进一步利用节点名称中的质量标识进行 Smart 筛选。

Smart 策略组根据节点名称、地区及质量等级进行更加精细化的节点分类。

其中 `PRM` 节点继续作为高质量线路使用，并针对香港、新加坡、日本等常用地区进行独立筛选。

部分 Smart 策略组进一步限定节点必须包含 `PRM` 标识，以避免低质量节点混入高质量线路。

---

## 十六、GitHub Ruleset 个人化定位

当前 GitHub Ruleset 中的规则均来自个人实际使用过程中的记录与调整。

这些规则与以下因素高度相关：

* 节点选择
* 网络地区
* ISP
* DNS
* 代理服务
* 个人使用习惯
* 实际访问情况

因此，本规则集并不以构建通用规则集为目标。

同一个域名在不同网络环境下可能具有完全不同的最佳策略。

因此，本项目中的代理、直连以及特殊分流结果仅代表当前个人网络环境下的选择。

**这些规则不保证适用于其他用户、网络环境、运营商、地区或代理配置。**

---

## 策略结构总结

当前整体策略结构主要包括：

* 多机场
* 多地区
* 多质量等级（PRM / STD / BSC）
* Smart Group 智能节点选择
* YouTube AdFree 专用策略组
* Emby 专用策略组
* Test IP 检测策略组
* AdBlock 广告拦截规则
* FavSites 常用网站规则集
* Direct 直连规则集
* GitHub Rule Provider
* 每小时自动更新规则

---

## Personal Use

**These rules are provided for personal use only; availability for other users is not guaranteed.**

The rules are collected and maintained according to actual usage, personal network conditions, preferred services, proxy nodes, ISP, DNS configuration, and individual requirements.

They are **not intended to be a universal ruleset**, and there is no guarantee that they will work correctly or provide the expected routing behavior in other environments.

Use these rules at your own discretion.

---

> 「應無所住而生其心」
>
> Last updated: **August 24, 2026**


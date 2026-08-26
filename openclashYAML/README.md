# Smart Group Config for OpenClash

一套以 **Smart 智能测速 + 节点质量分级 + 地区分类** 为核心的 OpenClash 配置方案。

本项目通过 **OpenClash Override 节点重命名预处理 + 正则过滤 + Smart 策略组**，将不同机场的节点统一标准化，实现多机场节点的自动分类、自动测速与智能切换。

> **核心理念：**
>
> **统一节点命名 → 按地区 / 等级自动分类 → Smart 自动测速 → 自动选择最优节点**

适合同时使用多个机场，并希望减少手动维护策略组、节点列表的用户。

---

## ✨ 核心特性

* 🧠 **Smart 智能测速**：自动选择当前延迟和质量更好的节点
* 🌏 **地区自动分类**：HK / JP / SG / US 等地区独立 Smart
* ⭐ **节点质量分级**：通过 `PRM` / `BSC` 区分优质与普通节点
* 🔄 **多机场统一管理**：不同机场的节点统一命名规则
* 🏷️ **机场来源识别**：通过节点前缀快速区分机场来源
* 📡 **自动故障切换**：节点不可用时自动切换其他可用节点
* 📚 **Rule-Providers**：规则集采用远程引用，方便更新
* 🗺️ **GEO 数据库**：结合 GEOSITE / GEOIP 处理常规国内外分流
* 🎨 **Zashboard UI**：提供配套的策略组图标和界面配置

---

## 🖥️ 运行环境

本项目主要在以下环境中测试：

| 项目   | 环境                             |
| ---- | ------------------------------ |
| 硬件   | Lunzn FastRhino R68S（电犀牛 R68S） |
| 固件   | ImmortalWrt 24.10.4            |
| 核心插件 | OpenClash                      |
| 推荐核心 | Mihomo                  |
| 面板   | Zashboard                      |

> 不代表只能运行于上述硬件和固件环境。
>
> 只要 OpenClash / Mihomo 对相关配置项提供支持，原则上均可使用。

---

# 🔑 核心机制：Override 节点标准化

这是整个配置能够自动运行的**关键步骤**。

不同机场的节点名称通常并不统一，例如：

```text
香港 01
HK-01
Hong Kong 01
HKG-01-专线
日本东京 01
JP-Tokyo-01
```

如果直接使用这些节点，Smart 策略组很难通过正则准确识别。

因此，本项目要求在 OpenClash 导入机场订阅时，通过 **Override** 对节点名称进行统一处理。

---

## 1. 节点前缀：识别机场来源

例如：

```yaml
override:
  additional-prefix: "机场B-"
```

最终节点名称：

```text
机场B-HK-01-PRM
机场B-HK-02-PRM
机场B-JP-01-BSC
```

这样可以在 Zashboard / OpenClash 节点列表中快速判断节点属于哪个机场。

例如：

```text
机场A-HK-01-PRM
机场A-HK-02-BSC

机场B-HK-01-PRM
机场B-JP-01-PRM
```

---

## 2. 节点后缀：区分节点等级

节点后缀是 Smart 策略组进行**质量分类**的重要依据。

### PRM — Premium

用于优质、高质量、专线等节点：

```text
-HK-01-PRM
-JP-02-PRM
-SG-03-PRM
```

### BSC — Basic

用于普通、基础节点：

```text
-HK-01-BSC
-JP-02-BSC
-SG-03-BSC
```

例如：

```text
机场A-HK-01-PRM
机场A-HK-02-BSC
机场B-JP-01-PRM
机场B-JP-02-BSC
```

这样策略组就可以通过正则自动筛选：

```text
PRM → 优质节点
BSC → 普通节点
```

**无需手动将节点加入策略组。**

---

## 3. 节点名称统一

通过 `proxy-name` 将不同机场使用的地区名称统一。

例如：

```yaml
override:
  proxy-name:
    - replace: "香港"
      into: "HK"

    - replace: "台湾"
      into: "TW"

    - replace: "日本"
      into: "JP"

    - replace: "新加坡"
      into: "SG"

    - replace: "美国"
      into: "US"
```

例如：

```text
香港 01
Hong Kong 01
HKG 01
```

经过处理后可以统一为：

```text
HK 01
```

这样地区 Smart 策略组就可以使用简单的地区关键词进行匹配。

---

# ⚙️ Override 完整示例

一个机场的 Override 可以类似这样配置：

```yaml
override:
  # 机场来源
  additional-prefix: "机场B-"

  # 节点等级
  additional-suffix: "-PRM"

  # 节点名称统一
  proxy-name:
    - replace: "香港"
      into: "HK"

    - replace: "台湾"
      into: "TW"

    - replace: "日本"
      into: "JP"

    - replace: "新加坡"
      into: "SG"

    - replace: "美国"
      into: "US"

    # 删除无用关键词
    # - replace: "需要删除的关键字"
    #   into: ""
```

最终可能得到：

```text
机场B-HK-01-PRM
机场B-HK-02-PRM
机场B-JP-01-PRM
机场B-SG-01-PRM
机场B-US-01-PRM
```

---

# 🧠 Smart Group 工作方式

完成 Override 标准化后，整个配置的工作流程如下：

```text
机场订阅
   ↓
Override
   ↓
统一节点名称
   ↓
地区 / PRM / BSC 自动识别
   ↓
正则 Filter
   ↓
Smart 策略组
   ↓
自动测速
   ↓
选择当前最优节点
   ↓
网络流量
```

用户只需要维护机场订阅和 Override，**不需要频繁手动调整节点。**

---

# 🚦 策略组结构

## 👑 1. 整体 PRM Smart

这是整个配置中的核心优选组之一。

它会自动收集所有机场中带有：

```text
-PRM
```

后缀的节点。

例如：

```text
机场A-HK-01-PRM
机场A-JP-01-PRM
机场B-SG-02-PRM
机场B-US-01-PRM
```

所有符合条件的节点都会进入整体 PRM Smart。

因此：

> 不需要关心节点属于哪个机场、哪个地区，只需要让 Smart 自动选择当前表现最好的 PRM 节点。

---

## 🌐 2. 默认 / 兜底 Smart

默认 Smart 组包含更广泛的可用节点。

主要用于：

* 默认代理出口
* 未单独指定地区的流量
* 没有特殊规则匹配的流量
* 作为其他策略组的兜底出口

---

# 🌏 3. 地区 Smart

地区 Smart 根据节点名称中的地区标识自动筛选节点。

例如：

### 🇭🇰 HK Smart

自动筛选：

```text
HK + PRM
HK + BSC
```

或根据实际策略只筛选指定等级。

### 🇯🇵 JP Smart

```text
JP + PRM
JP + BSC
```

### 🇸🇬 SG Smart

```text
SG + PRM
SG + BSC
```

### 🇺🇸 US Smart

```text
US + PRM
US + BSC
```

其他地区可以按照相同方式扩展。

---

# 🎯 Smart 的优势

与手动指定节点相比，Smart 策略组最大的优势是：

```text
节点增加
   ↓
自动进入对应节点池
   ↓
自动测速
   ↓
自动淘汰不可用节点
   ↓
自动选择表现更好的节点
```

例如：

```text
机场A-HK-01-PRM
机场A-HK-02-PRM
机场B-HK-01-PRM
机场B-HK-02-PRM
```

不需要手动判断哪个节点最快。

Smart 会根据实际网络情况进行选择。

---

# 📡 规则分流

本配置采用：

**Rules → Proxy Groups → Smart**

的方式进行流量分流。

例如：

```text
ChatGPT
   ↓
AI 服务策略组
   ↓
US / SG / JP Smart
   ↓
自动选择最优节点
```

或者：

```text
Netflix
   ↓
流媒体策略组
   ↓
指定地区 Smart
   ↓
自动选择最优节点
```

对于普通代理流量，则可以直接使用：

```text
整体 PRM Smart
```

---

# 📚 规则系统

为了降低 YAML 的维护成本，本项目没有将大量规则直接写入配置文件，而是采用多种规则来源组合。

## Rule-Providers

大量规则通过远程 Rule-Providers 获取，例如：

* 广告拦截
* 流媒体
* AI 服务
* 国内 / 国外域名
* IP 地址段
* 其他第三方规则集

这样可以避免频繁修改主配置文件。

---

## GEO 数据库

常规国内外流量使用 Mihomo / Meta 内置 GEO 数据库处理。

例如：

```yaml
GEOSITE,cn,DIRECT
GEOIP,cn,DIRECT
```

适用于大量常规国内外网站的快速分流。

---

## Inline Rules

只有少量需要高优先级处理的规则直接写入：

```yaml
rules:
```

例如：

* 局域网流量
* 自建服务
* 特殊域名
* 必须优先匹配的规则

这样可以让主配置保持相对简洁。

---

# 🎨 Zashboard

本项目同时提供配套的：

```text
zashboard-settings.json
```

用于优化 Zashboard 的显示效果。

主要包含：

* 策略组 Icons
* 策略组布局
* 卡片显示
* 项目专属 UI 配置

---

## 导入方式

1. 打开 Zashboard
2. 进入 **Settings / 设置**
3. 找到配置导入 / 备份恢复功能
4. 导入：

```text
zashboard-settings.json
```

5. 刷新 Zashboard

即可获得与本项目策略组结构匹配的界面。

---

# ⚠️ 添加新机场时

这是使用本项目时最需要注意的一点。

**每添加一个新的机场订阅，都需要设置对应的 Override。**

至少需要考虑：

```text
机场前缀
+
节点地区名称
+
节点等级后缀
```

例如：

```text
机场B-HK-01-PRM
机场B-JP-01-PRM
机场B-SG-01-BSC
机场B-US-01-BSC
```

---

## ❌ 不推荐

如果节点最终名称为：

```text
机场B-HK-01
机场B-JP-01
机场B-SG-01
```

由于缺少：

```text
-PRM
```

或：

```text
-BSC
```

对应的 Smart Filter 可能无法正确识别节点。

---

## ✅ 推荐

统一为：

```text
机场B-HK-01-PRM
机场B-HK-02-PRM
机场B-JP-01-BSC
机场B-SG-01-BSC
```

这样新增节点后即可自动进入对应的 Smart 节点池。

---

# 📌 推荐命名规范

建议所有机场最终统一成：

```text
[机场名称]-[地区]-[节点编号]-[等级]
```

例如：

```text
机场A-HK-01-PRM
机场A-HK-02-PRM
机场A-JP-01-BSC

机场B-SG-01-PRM
机场B-US-01-PRM
机场B-US-02-BSC
```

其中：

| 字段   | 示例    | 作用          |
| ---- | ----- | ----------- |
| 机场名称 | `机场A` | 区分节点来源      |
| 地区   | `HK`  | 地区 Smart 分类 |
| 节点编号 | `01`  | 区分具体节点      |
| 等级   | `PRM` | 节点质量分类      |

---

# 🚀 总结

本项目并不是单纯提供一份 OpenClash YAML，而是建立了一套完整的：

```text
节点标准化
    ↓
机场来源识别
    ↓
地区分类
    ↓
节点质量分级
    ↓
Smart 自动测速
    ↓
规则自动分流
```

体系。

其中 **Override 是整个系统的基础**。

只要新增机场时按照统一规则处理节点名称，后续的：

* 地区 Smart
* PRM Smart
* BSC 节点池
* 默认 Smart
* AI / 流媒体策略
* 自动测速
* 故障切换

都可以自动完成，最大程度减少日常维护成本。

> **推荐做法：**
>
> **机场负责提供节点，Override 负责标准化，Filter 负责分类，Smart 负责选择。**

# Smart Group Release Version Lite

基于 **OpenClash + Mihomo Smart** 的轻量版配置模板。

核心思路是使用 **Smart 策略组 + LightGBM** 自动选择节点，并按照地区对节点进行分类，同时通过节点名称后缀区分不同等级的机场节点。

## 特点

* **Smart 策略为主**
* 按地区自动划分节点
* 支持香港、日本、新加坡、美国地区 Smart 组
* 提供整体 `SMART` Smart 组
* 使用 **LightGBM** 自动选择节点
* 支持通过节点后缀区分机场等级
* `PRM` 节点设置更高优先级
* 内置常用网站、流媒体、游戏平台及 AI 服务规则
* 内置广告拦截规则
* 规则集通过 GitHub 自动更新
* 相比完整版本更加精简，适合日常使用

## Smart 策略组

| 策略组             | 用途                    |
| --------------- | --------------------- |
| `SMART`         | 所有 PRM 节点的综合 Smart 选择 |
| `HKG`           | 香港地区节点                |
| `JPN`           | 日本地区节点                |
| `SGP`           | 新加坡地区节点               |
| `USA`           | 美国地区节点                |
| `HOME`          | 家宽 / 家庭 / 住宅节点        |
| `Manual Choice` | 手动选择节点                |
| `Download`      | 下载 / BT 等流量           |

地区 Smart 组会根据节点名称自动匹配对应地区，无需手动维护节点列表。

## 节点命名

配置通过节点名称中的前缀和后缀区分不同机场及节点等级。

例如：

```text
机场A-香港01-PRM
机场A-日本01-PRM
机场B-香港01-BSC
机场B-新加坡01-BSC
```

其中：

* `机场A` / `机场B`：用于区分不同机场
* `PRM`：高优先级节点
* `BSC`：普通节点

`PRM` 节点在 Smart 策略中的优先级设置为 `1.3`。

## 规则

配置针对常用服务进行了独立策略分流，包括：

* Instagram / Facebook / Threads
* ChatGPT
* Gemini
* Google
* GitHub
* Telegram
* YouTube
* Netflix
* TikTok
* Spotify
* Steam
* Apple
* Microsoft
* 游戏平台
* Emby
* TVB
* 广告拦截

其中：

* **Facebook、Instagram 和 Threads 均使用 Instagram 策略**
* **Gemini 使用 Google 策略**

## 使用

1. 下载 YAML 配置文件。
2. 在 `proxy-providers` 中填入自己的机场订阅链接。
3. 根据实际情况修改机场名称及节点后缀。
4. 导入 OpenClash。
5. 根据自己的需求修改规则和策略组。

机场订阅示例：

```yaml
additional-prefix: "机场A-"
additional-suffix: "-PRM"
```

配置文件中的机场订阅部分已经预留好位置，只需要填入订阅链接即可。

## 注意

此配置主要针对 **Mihomo / OpenClash Smart** 使用。

`lite` 版本以实用和简洁为主，不追求加入大量额外功能；如果需要进一步修改，可以直接在本地调整 YAML。

---

**配置文件：**

[Smart Group Release Version lite.yaml](https://github.com/chungccc/yaml-config/blob/main/openclashYAML/Smart%20Group%20Release%20Version%20lite.yaml)

**最后更新：** 2026-08-24


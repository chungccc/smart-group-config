Smart Group Config for OpenClash
本项目提供了一套高度自动化、基于延迟优选（Smart）与节点质量分级的 OpenClash 配置文件。通过 OpenClash 的 override 重命名预处理以及精细的正则过滤，实现多机场节点的格式统一与网络流量的智能无感切换。

🖥️ 运行环境
本套配置在以下软硬件环境下测试并稳定运行：

硬件设备: Lunzn FastRhino R68S (电犀牛 R68S)

固件版本: ImmortalWrt 24.10.4

核心插件: OpenClash (建议配合 Meta/Mihomo 核心)

控制面板: Zashboard (搭配本项目专属的 zashboard-settings.json 实现定制化图标与 UI 配置)

🔑 核心关键：通过 override 实现节点标准化
多机场订阅的节点命名规则千奇百怪（例如有的叫 香港 01，有的叫 HK-01-专线）。为了让本配置中的 Smart 策略组能够精准提取节点，必须通过 override 机制在导入订阅时对节点名称进行统一预处理。

这是整个配置文件能够自动化运作的最核心前提！

示例 override 配置：
YAML
override:
  additional-prefix: "机场B-"  # 1. 前缀：用于标识节点来源（方便在节点列表中快速区分不同机场）
  additional-suffix: "-BSC"   # 2. 后缀：用于标记节点等级（关键！供策略组正则匹配）
  proxy-name:                 # 3. 关键字替换/删除：统一节点命名格式（可按需增删）
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
    # - replace: "需要删除的无用关键字"
    #   into: ""
💡 为什么 override 如此重要？
统一等级后缀 (additional-suffix)：

优质机场/专线节点：赋予 -PRM (Premium) 后缀。

普通/基础机场节点：赋予 -BSC (Basic) 后缀。

策略组通过识别 -PRM 或 -BSC，自动把节点分流到“优质组”或“基本组”，无需手动挑节点。

统一地区标识 (proxy-name)：

将不同机场的 香港、HongKong、HKG 等统一定换为 HK。

确保地区 Smart 组（如 HK Smart）可以通过简洁的正则（如 HK|香港）无遗漏地抓取所有该地区的节点。

识别机场来源 (additional-prefix)：

加上 机场A-、机场B- 前缀，方便在面板中排查特定机场的节点连通性。

⚙️ 配置运作方式 (How it works)
在通过 override 完成节点名称的标准化后，本配置的流量分发逻辑如下：

规则匹配 (Rules)：流量进入路由，匹配到对应的规则（如流媒体、AI 服务、特定国家等）。

策略选择 (Proxy Groups)：规则指向指定的策略组，绝大部分核心流量会被指向 整体 PRM SMART 组 或 地区 SMART 组。

Smart 自动优选 (url-test)：

Smart 组（url-test）会定时对池内的节点进行高频测速。

自动过滤掉不可用/超时节点，并将流量平滑切换到当前延迟最低、最稳定的节点上。

质量过滤 (Filter)：

整体 PRM SMART 组：通过正则只抓取带 -PRM 后缀的节点，保证极致速度与稳定性。

地区 SMART 组：结合地区关键字与等级后缀，优先调用该地区的优质节点。

🗂️ 策略组 / 节点组展示
在 Zashboard 面板中，您将看到以下层次分明的策略组布局：

1. 核心 Smart 优选组
👑 整体 PRM 节点 SMART 组：全局最高优先级的智能测速组。汇聚了所有机场带 -PRM 后缀的优质节点。无论哪个地区的专线最快，流量就自动走哪里。

🌐 默认优选 / 兜底 Smart 组：包含所有可用节点，作为默认未命中规则时的智能出口。

2. 地区 SMART 组 (Regional Smart)
按地区划分的独立 Smart 测速池，结合了地区与等级识别：

🇭🇰 HK Smart (香港优选) -> 仅在带有香港标识且符合后缀要求的节点中优选

🇯🇵 JP Smart (日本优选)

🇸🇬 SG Smart (新加坡优选)

🇺🇸 US Smart (美国优选)

(其他地区依此类推...)

3. 特殊业务组
🍎 Apple 服务：针对苹果生态的规则组，支持直连或指定低延迟池。

🤖 AI 服务 / 国际流媒体：针对 ChatGPT/Claude/Netflix 等限制严格的服务，定位到原生或解锁质量最好的特定节点池。

📜 规则引用方式介绍
为了保证规则的实时性与轻量化，本配置采用了以下组合方案：

Rule-Providers (动态规则集)：
绝大部分规则不硬编码在 YAML 中，而是通过 rule-providers 动态拉取远程列表（如大中华区 IP、广告拦截列表、各家流媒体域名等），支持热更新。

内置 GEO 数据库 (GEOSITE / GEOIP)：
结合 Meta 核心内置的 GEO 数据库，高效处理常规的国内外分流（如 GEOSITE,cn,DIRECT ）。

自定义直写规则 (Inline Rules)：
仅在 rules: 顶部保留极少数高优先级的自定义需求（如局域网透传、自建服务直连等）。

🎨 面板配置指引 (Zashboard)
为了获得最佳的视觉体验和操作逻辑，本项目配套使用了 Zashboard 面板。

导入步骤：

本仓库提供了 zashboard-settings.json 配置文件（包含了定制的策略组图标 Icons 与卡片布局）。

打开 Zashboard 面板，进入 设置 (Settings) 菜单。

找到配置导入/备份恢复功能，选择并导入 zashboard-settings.json。

刷新页面，即可呈现与本 YAML 完全匹配的图标化策略组界面。

💡 重要提醒
导入新机场时必须配置 override：
每当您在 OpenClash 中添加一个新的机场订阅时，请务必按照上述 override 格式设置其前缀、后缀和重命名规则。如果后缀不包含 -PRM 或 -BSC，节点将无法被对应的 Smart 策略组抓取！

---
layout: default
title: 基于KiCad的大语言模型插件(Agent) — 解决Windows版本卡死问题
tags: [LLM, MCP, KiCad, Windows]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 基于KiCad的大语言模型插件(Agent) — 解决Windows版本卡死问题


## 前言

由于着急发布版本 1.0.0 ，我仅对 Windows 版本做了简单测试。后来发现在 Windows 平台上 WebView 组件的刷新方法导致插件很容易卡死，几乎无法使用。由于 Windows 上 WebView 组件提供的是异步接口，原刷新方法容易造成死锁。经过反复尝试，重构聊天历史窗口的刷新方法后才解决该问题。

新的版本 1.1.1 解决了下列问题：

1.  修复了 Windows 平台下插件界面偶发卡死/无响应的问题（尤其是在展示富文本或交互内容时），改善了稳定性和出错提示。
2.  若干 UI 改进：聊天窗口在 AI 输出时保持固定，恢复后可靠滚动到最新消息。

同时新增了下列功能：

1.  **分组布局工具** ：支持对元器件管脚进行分组，分别优化布局。支持整体移动和旋转分组。
2.  **PCB Zone 管理工具** ：支持创建和删除填充区与 keepout 区域。
3.  插件在更新原理图或 PCB 前会自动创建快照。修改 PCB 后会刷新 PCB 编辑器；但受 IPC 限制，目前无法自动刷新原理图编辑器。

下载地址见参考索引中 Releases 页面。

相关文章：

1.  [如何使用LLM直接修改KiCad原理图](https://paul356.github.io/2026/04/09/kicad-mcp.html)
2.  [基于KiCad的大语言模型插件(Agent)](https://paul356.github.io/2026/04/24/kicad-plugin.html)
3.  [基于KiCad的大语言模型插件(Agent) - 支持编辑PCB](https://paul356.github.io/2026/05/08/kicad-plugin-pcb.html)
4.  [基于KiCad的大语言模型插件(Agent) - 支持自动布线](https://paul356.github.io/2026/05/11/kicad-plugin-autoroute.html)
5.  [基于KiCad的大语言模型插件(Agent) - Windows平台安装指南](https://paul356.github.io/2026/05/20/kicad-plugin-windows.html)


## 新功能介绍


### **分组布局工具**

| # | 工具名                  | 作用                   |
|--- |----------------------- |---------------------- |
| 1 | `assign_to_group`       | 批量将封装标记到指定组 |
| 2 | `list_groups`           | 列出 PCB 上所有组与成员统计 |
| 3 | `get_group`             | 获取某个组的详细信息（成员/锚点/bbox） |
| 4 | `score_group`           | 计算组内布局质量指标（HPWL） |
| 5 | `place_component_group` | 优化组内的元器件布局，并找到合适位置放置该组 |
| 6 | `move_group`            | 平移已放置的组（刚体移动） |
| 7 | `rotate_group`          | 旋转已放置的组（围绕锚点） |

为了优化元器件的布局流程，我设计了元器件分组、组内布局优化、移动和旋转等工具，采用分治的方法来确定元器件的空间布局。用户可以通过指导大语言模型对 PCB 元器件进行分组，再分别优化组内布局，最后再整体放置元器件组到合适的位置。该方式比直接让大模型来决定元器件的位置更加高效，成功率更高。

可先让大语言模型使用 `assign_to_group` 工具将元器件分组。

![img](/images/kicad-plugin-group-pcb-items.png)

然后让大语言模型使用 `place_component_group` 分别优化各元器件组的组内布局。

![img](/images/kicad-plugin-optimize-item-groups.png)

等各元器件组内布局优化完成后，将各元器件组移动到PCB上的合适位置。

![img](/images/kicad-plugin-group-optimize-result.png)

最后使用自动布线（Auto Route）完成布线。

![img](/images/kicad-plugin-group-optimize-route.png)


### **PCB Zone 管理工具**

| # | 工具名         | 作用                                             |
|--- |-------------- |------------------------------------------------ |
| 1 | `list_zones`   | 列出 PCB 上所有 zones（copper\_pour / keepout，即填充区/禁区） |
| 2 | `add_zone`     | 添加新的铜皮或 keepout 区域（即禁区）            |
| 3 | `delete_zone`  | 按 UUID 删除指定的 zone                          |
| 4 | `refill_zones` | 重填所有的填充 zone                              |

为支持填充和 Keepout 区域我增加了创建、删除和填充 Zone 的工具，可允许大语言模型创建 GND 等铜皮区域。

![img](/images/kicad-plugin-fill-zone-request.png)

![img](/images/kicad-plugin-fill-zone-result.png)


## 总结和计划

本文介绍了新版本 KiCad AI Assistant，解决了 Windows 平台卡死问题，并新增布局分组优化及 Zone 管理工具。下一步考虑添加的功能有：

1.  PCB设计约束管理功能
2.  元器件成本管理功能
3.  自动生成原理图符号和PCB元器件管脚

工具仍处于早期阶段，但随着功能持续增加和完善，相信逐渐能对 PCB 设计有所帮助。欢迎大家提出反馈，也欢迎有兴趣的朋友参与设计与开发。为了方便大家反馈意见，我创建了一个微信群，有兴趣的朋友可以加入群聊交流技术。

![img](/images/kicad-plugin-wexin-group-chat.jpg)


## 参考链接

1.  kicad-mcp 项目 - <https://github.com/paul356/kicad-mcp>
2.  发布记录（Releases） - <https://github.com/paul356/kicad-mcp/releases>

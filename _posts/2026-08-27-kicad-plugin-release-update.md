---
layout: default
title: 基于KiCad的大语言模型插件(Agent) — 0.2.1 更新：会话按项目切换与稳定性修复
tags: [KiCad, MCP, plugin, Session, Streaming, Router]
nav_order: 2026-08-27
sync_wexin: 1
---


# 基于KiCad的大语言模型插件(Agent) — 0.2.1 更新：会话按项目切换与稳定性修复


## 前言

本次更新（v0.2.1）主要是修复我在日常使用 KiCad AI Assistant 过程中遇到的一批实际问题：会话和项目不绑定、切换项目时面板行为异常、LLM 长回复超时、工具调用失败后板子视图不刷新、布线到金手指焊盘失败等。这些问题不影响插件核心功能，但确实影响使用体验和稳定性，这次集中修复后顺手了不少。

本系列相关文章：

1.  [如何使用LLM直接修改KiCad原理图](https://paul356.github.io/2026/04/09/kicad-mcp.html)
2.  [基于KiCad的大语言模型插件(Agent)](https://paul356.github.io/2026/04/24/kicad-plugin.html)
3.  [基于KiCad的大语言模型插件(Agent) - 支持编辑PCB](https://paul356.github.io/2026/05/08/kicad-plugin-pcb.html)
4.  [基于KiCad的大语言模型插件(Agent) - 支持自动布线](https://paul356.github.io/2026/05/11/kicad-plugin-autoroute.html)
5.  [基于KiCad的大语言模型插件(Agent) - Windows平台安装指南](https://paul356.github.io/2026/05/20/kicad-plugin-windows.html)
6.  [基于KiCad的大语言模型插件(Agent) — 解决Windows版本卡死问题](https://paul356.github.io/2026/05/26/kicad-plugin-hang-issue.html)
7.  [基于KiCad的大语言模型插件(Agent) — 简化安装方法](https://paul356.github.io/2026/05/29/kicad-plugin-support-pypi.html)
8.  [基于KiCad的大语言模型插件(Agent) — 特性更新：Skills、嵌套电路编辑及更多](https://paul356.github.io/2026/06/05/kicad-plugin-feature-update.html)
9.  [基于KiCad的大语言模型插件(Agent) — DRC设计规则与自动布线联动](https://paul356.github.io/2026/06/11/kicad-plugin-drc-freeroute.html)
10. [基于KiCad的大语言模型插件(Agent) — 支持 PNS 避障布线](https://paul356.github.io/2026/07/29/kicad-plugin-pns-router.html)
11. [基于KiCad的大语言模型插件(Agent) — 支持 Vision 模型与 PDF 文本提取](https://paul356.github.io/2026/08/07/kicad-plugin-vision-pdf.html)


## 更新范围

本次更新（v0.2.1）合并了以下变更：

-   会话按项目持久化：会话与 `.kicad_pro` 项目文件绑定，面板打开时自动恢复当前项目的会话，切换项目时原地切换会话，不再残留窗口或串项目
-   LLM 响应流式输出：KiCad 内置 Python 缺少 `ssl` 模块时的回退路径改为流式转发，非流式调用超时统一从 60 秒提高到 300 秒，长回复不再超时
-   工具调用失败后刷新板子视图：布线等工具失败时也能触发 KiCad 文档重载，不再出现界面提示已刷新但板子实际没变的假象
-   布线修复：支持布线到同名焊盘（如金手指）和板边安装焊盘，单层 A\* 按铜层过滤障碍物，并通过转弯惩罚减少布线拐弯与分段
-   封装 Edge.Cuts 几何可通过 PCB 查询工具读取
-   版本号升级至 `0.2.1`


## 功能详解


### 会话按项目持久化与切换

此前会话在退出时自动保存，且不区分项目，经常出现这次打开的会话跑到另一个项目里、切换项目后面板还停留在旧会话上的问题。这个版本将会话持久化逻辑单独抽出：

-   会话在保存时记录所属的 `.kicad_pro` 项目路径（存储 schema v2），并在每次发送、回复和取消时落盘，重启 KiCad 也不会丢失已完成的消息
-   面板首次显示时自动恢复当前打开项目的会话；因为 KiCad 没有提供项目切换事件，插件每 1 秒轮询一次项目路径、连续两次确认后再原地切换会话，面板保持打开而不是对顶层窗口调用 `Destroy()` 把 KiCad 的工程窗口一并关掉
-   “加载会话”对话框只列出当前项目的会话，并顺带修复了面板在后端就绪前显示时报 `unknown url type: 'None/mcp'` 的错误


### 工具调用失败后刷新板子视图

使用中发现 `pcb_route_pad_to_pad` 等修改类工具出错时，板子视图不会刷新，但界面上照样显示 "Board view refreshed."。原因是失败路径提前返回，没有把修改过的文件标记为 dirty，而刷新提示是按文件名显示的、照样会触发。现在失败路径同样标记 dirty 路径，并统一在一个 `try/finally` 收尾逻辑里覆盖所有退出路径（包括 LLM 报错、迭代上限、异常退出），刷新失败也不会掩盖本轮结果。


### LLM 响应流式输出与超时统一

KiCad 内置的 Python 没有 `ssl` 模块，之前带 SSL 回退的调用会降级成一次性的非流式子进程 POST，60 秒超时对大模型长回复来说不够用，经常话还没说完就断了。现在子进程直接解析 SSE 流，把增量逐块转发给界面，保持流式体验；非流式调用的超时也统一提高到 300 秒。


### 布线到同名焊盘与板边安装焊盘

布线工具之前连不上实际板上常见的几种焊盘：

-   同名焊盘：在一块板子上给 J2/3 → U11/3v3 布线时，遇到的 micro:bit 金手指连接器有 4 个同名 `3v3` 焊盘（3 个 SMD 金手指 + 1 个过孔焊盘），旧的按名字精确匹配的逻辑处理不了
-   板边安装焊盘：连接器槽位的 `Edge.Cuts` 画在封装内部（~fp\_line~ / `fp_rect~），旧代码只收集顶层 ~gr_*` 外框，把连接器位置误判为板外，直接拒绝布线

修复后：板框识别递归进入封装内部、把 `fp_*` 几何变换到世界坐标；同名网络过孔焊盘的钻孔、焊盘铜皮全部恢复为障碍物（末端焊盘除外）；过孔禁止区覆盖所有同名网络焊盘；新增 `layer_hint` 参数自动选层；A\* 加转弯惩罚，实测同一根线从 12 段降到 9 段再到 2 段（一条 45° 加一条竖线，无过孔）。


### 封装 Edge.Cuts 几何查询

PCB 查询工具现在可以读取封装内部的 `Edge.Cuts` 几何，布线工具也依赖它识别连接器槽位等特殊外形。


## 结语

这个版本的问题大多来自我实际画板子时的真实场景，比如金手指布线、切换项目时面板被关掉、长回复超时这些，经过这次修复之后日常使用稳定了很多。如果你在插件使用中遇到其它问题，欢迎提 Issue 或者 PR，也欢迎加入技术交流群交流意见。升级后如果发现行为没有变化，请彻底重启一次 KiCad 让新版代码生效。

![img](/images/kicad-plugin-wexin-group-chat5.png)


## 参考

-   代码仓库：<https://github.com/paul356/KiCad-AI-Assistant>
-   PyPI 包：<https://pypi.org/project/kcaa/>
-   0.2.1 发布页面：<https://github.com/paul356/KiCad-AI-Assistant/releases/tag/0.2.1>

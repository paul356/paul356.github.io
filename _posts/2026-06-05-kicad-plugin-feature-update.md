---
layout: default
title: 基于KiCad的大语言模型插件(Agent) — 特性更新：Skills、嵌套电路编辑及更多
tags: [KiCad, MCP, plugin, 特性更新]
nav_order: 2026-06-05
sync_wexin: 1
---


# 基于KiCad的大语言模型插件(Agent) — 特性更新：Skills、嵌套电路编辑及更多

本系列相关文章：

1.  [如何使用LLM直接修改KiCad原理图](https://paul356.github.io/2026/04/09/kicad-mcp.html)
2.  [基于KiCad的大语言模型插件(Agent)](https://paul356.github.io/2026/04/24/kicad-plugin.html)
3.  [基于KiCad的大语言模型插件(Agent) - 支持编辑PCB](https://paul356.github.io/2026/05/08/kicad-plugin-pcb.html)
4.  [基于KiCad的大语言模型插件(Agent) - 支持自动布线](https://paul356.github.io/2026/05/11/kicad-plugin-autoroute.html)
5.  [基于KiCad的大语言模型插件(Agent) - Windows平台安装指南](https://paul356.github.io/2026/05/20/kicad-plugin-windows.html)
6.  [基于KiCad的大语言模型插件(Agent) — 解决Windows版本卡死问题](https://paul356.github.io/2026/05/26/kicad-plugin-hang-issue.html)
7.  [基于KiCad的大语言模型插件(Agent) — 简化安装方法](https://paul356.github.io/2026/05/29/kicad-plugin-support-pypi.html)


## 概要

有几天没有更新了，我中间去开发了一个工具 copilot-anywhere ，可以让程序员遛狗时也能开发代码，有兴趣的的朋友可以去看我上一篇公号文章。本次 KiCad AI Assistant 插件更新到 v0.1.5，带来两个重要的新特性：

-   **Skills 系统** ：AI 会根据你的需求自动加载对应的操作指南，减少系统提示的篇幅，节省 Token 消耗，也让 AI 的回复更精准、响应更快。
-   **嵌套电路编辑** ：现在 AI 可以帮你管理多页原理图的层次结构——创建子页、修改子页、添加层次引脚等操作都可以通过对话完成。

此外，本次更新还包括多项 UI 改进、布线策略优化、跨文件元件引用检查，以及 Windows 平台的兼容性修复。

![img](/images/kicad-plugin-ui-enhancements.png)


## 1. Skills 系统

Skills 系统允许用户定制 Skill 文件来指导大模型完成任务。


### 什么是 Skill 文件

Skill 文件就是给大模型准备的操作指南，采用 markdown 格式撰写，用于告诉大模型一些特殊的注意事项、经验准则和特殊要求等内容。大模型会根据任务的相关性读取相关的 Skill 文件。

新版本把这些操作指南拆成了 **6 个独立的 Skill 文件** ，每个文件针对一种常见任务，用户也可以自己增加新的 Skill 文件：

| Skill 名称              | 文件名                     | 适用场景             |
|----------------------- |-------------------------- |-------------------- |
| `schematic-placement`   | `schematic_placement.md`   | 在原理图中放置和排列元件 |
| `schematic-wiring`      | `schematic_wiring.md`      | 智能布线：连接引脚、避免交叉 |
| `pcb-query`             | `pcb_query.md`             | 查询 PCB 信息：元件列表、飞线、板框 |
| `pcb-placement`         | `pcb_placement.md`         | PCB 元件布局：放置、对齐、分布 |
| `pcb-outline`           | `pcb_outline.md`           | 编辑 PCB 板框外形    |
| `pcb-footprint-library` | `pcb_footprint_library.md` | 搜索和查看封装库     |


### 使用方式

插件会自动管理 Skill 的加载——当你说"帮我把电容放在 U1 右边"时，AI 会自动拉取 `schematic-placement` Skill 获取放置策略，而不需要你手动操作。你也可以在对话中直接说"帮我查询 PCB 上所有未连接的飞线"，AI 会加载 `pcb-query` Skill 来获取需要的工具和步骤。

Skill 文件位于插件安装目录的 `kicad_plugin/skills/` 子目录下，每篇都是一个 Markdown 文件。如果你有特定的电路设计规范或常用操作流程，可以仿照现有文件格式自行创建新的 `.md` 文件放入该目录——插件启动时自动识别，AI 即可在需要时加载你的自定义 Skill。

插件目录位置因平台而异：

-   Linux： `/.local/share/kicad/10.0/scripting/plugins/kicad_ai_assistant/`
-   Windows： `%USERPROFILE%\Documents\KiCad\10.0\scripting\plugins\kicad_ai_assistant\`

Skills 文件位于上述目录下的 `kicad_plugin/skills/` 子目录中。


## 2. 嵌套电路编辑 —— 用 AI 管理多页原理图

之前的插件只能操作单个页面，无法在工程中创建或管理嵌套电路（Hierarchical Sheet）。本次更新新增了以下操作能力：


### 创建新的嵌套电路

你可以直接对 AI 说出需求，例如：

```
帮我创建一个 Power Supply 子页，放在主原理图的右边，宽 200mm 高 150mm，同时生成子文件。
```

AI 会调用 `add_sheet_symbol` ，在当前位置插入一个 sheet 实例，并自动生成对应的 `power.kicad_sch` 子文件。你还可以同时指定层次引脚——例如在子页右侧添加 `VCC` 和 `GND` 引脚——AI 会把引脚自动放到 sheet 边缘的正确位置，无需手动绘制。

子文件创建后，你可以继续让 AI 往里面放置元件、连线——和操作普通原理图一样。


### 引用已有的原理图文件

如果你已经有现成的子原理图文件（例如从其他工程复制过来的 `mcu.kicad_sch` ），也可以让 AI 基于它创建嵌套电路。先把原理图文件拷贝到项目目录，然后对 AI 说：

```
帮我把 mcu.kicad_sch 作为子页添加到主原理图左侧，宽 200mm 高 150mm。
```

AI 会调用 `add_sheet_symbol` 并将 `sheet_file` 参数指向你指定的现有文件。


### 查看与管理

创建子页后，你可以随时让 AI 查看工程结构：

```
帮我看看整个工程的页面层次结构。
```

AI 会调用 `get_sheet_hierarchy` 返回完整的嵌套树。你也可以让 AI 修改子页的位置、大小、名称，或者添加/删除层次引脚——全部通过对话完成，无需手动操作 KiCad 界面。

此外，本次更新还增强了层次标签和全局标签的支持，以及跨文件元件引用冲突检测——如果不同页面的元件编号有冲突，AI 会帮你发现并修正。


## 3. 其他改进与修复

除了两大新特性，本次更新还包含多项用户可见的优化：


### UI 与交互

-   **会话区搜索** ：对话历史现在支持全文搜索，方便在长对话中回溯之前的操作。
-   **Stop 中断按钮** ：AI 回复过程中可以随时按下 Stop 中断生成，避免等待冗长的错误输出。
-   **工具结果折叠** ：每次 AI 调用的底层工具执行结果默认折叠显示，界面更清爽，需要时点击展开查看详情。


### 原理图编辑增强

-   **元件编号管理** ： `rename_symbol` 支持省略目标编号——AI 会自动从当前编号范围中分配下一个可用编号，不再需要手动指定。同时新增跨文件编号冲突检测（ `check_reference_conflicts` ），避免不同页面的元件编号重复。
-   **Netlist 增强** ： `extract_schematic_netlist` 现在会输出每个引脚的 **引脚名称** 和 **电气类型** （input/output/bidirectional 等），方便 AI 更准确地分析电路连接。
-   **全部标签支持** ：现在同时支持全局标签（Global Label）和层次标签（Hierarchical Label）的增删操作。
-   **冗余连线修复** ：修复了部分情况下连线重复或拓扑检测不准确的问题。


### 放置策略优化

-   元件放置时优先使用用户指定的目标位置，只有在检测到冲突时才微调——相比之前版本总是自动移位，现在更尊重你的意图。同时引入了最近优先的网格扫描算法，让冲突处理更快更准确。


### Windows 兼容性

-   修复了 Windows 上的文件编码（统一 UTF-8）和路径分隔符问题。
-   `setup_plugin.ps1` 增强了 KiCad 安装路径的自动搜索范围，覆盖更多非默认安装的情况。
-   安装超时时间从 30 秒延长到 60 秒，适应配置较低的机器。


## 4. 如何升级


### 第一步：下载插件包

从 [Releases 页面](https://github.com/paul356/KiCad-AI-Assistant/releases) 下载最新版 `kicad-ai-assistant.zip` ，解压到 KiCad 插件目录：

-   Linux： `/.local/share/kicad/10.0/scripting/plugins`
-   Windows: `%USERPROFILE%\Documents\KiCad\10.0\scripting\plugins`


### 第二步：运行安装脚本

进入插件目录，执行对应平台的安装脚本：

Linux：

```bash
cd ~/.local/share/kicad/10.0/scripting/plugins/kicad_ai_assistant
./setup_plugin.sh
```

Windows（PowerShell）：

```powershell
cd "$env:USERPROFILE\Documents\KiCad\10.0\scripting\plugins\kicad_ai_assistant"
.\setup_plugin.bat
```

安装脚本会自动从 PyPI 安装最新版 `kcaa` 包。如果你在国内网络环境下安装， `pip` 默认的国内镜像可能尚未同步 v0.1.5，可以设置环境变量指定 PyPI 官方源：

```bash
PIP_INDEX_URL=https://pypi.org/simple/ ./setup_plugin.sh
```

如果你之前安装的是 kicad-mcp 仓库的旧版插件，请参考 [简化安装方法](https://paul356.github.io/2026/05/29/kicad-plugin-support-pypi.html) 一文完成数据迁移。


## 结语

本次更新（v0.1.5）把 AI 的操作指南模块化为按需加载的 Skills，并把多页原理图的层次编辑交给 AI 与工具链处理，让复杂工程中的自动化编辑更加可靠与可控。欢迎在 GitHub 提交 Issue 或 PR，也欢迎加入技术交流群反馈意见。

![img](/images/kicad-plugin-wexin-group-chat1.png)


## 5. 参考

-   Skills 设计文档：<https://github.com/paul356/KiCad-AI-Assistant/blob/main/docs/skill-system-design.md>
-   Sheet 工具设计文档：<https://github.com/paul356/KiCad-AI-Assistant/blob/main/docs/sheet-crud-plan.md>
-   Skill 文件（欢迎贡献）：<https://github.com/paul356/KiCad-AI-Assistant/tree/main/kicad_plugin/skills>
-   代码仓库：<https://github.com/paul356/KiCad-AI-Assistant>
-   PyPI 包：<https://pypi.org/project/kcaa/>

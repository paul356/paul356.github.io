---
layout: default
title: 基于KiCad的大语言模型插件(Agent) - 支持编辑PCB
tags: [LLM, MCP, KiCad, PCB]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 基于KiCad的大语言模型插件(Agent) - 支持编辑PCB


## 前言

这一篇文章介绍我们对KiCad AI Assistant插件的增强，这些优化依然是针对Linux平台KiCad 10.0。相对于上一篇文章时的状态，这段时间我添加了PCB元器件相关的工具。

1.  支持搜索、查找系统目录中的元器件。
2.  支持设置原理图符号对应的PCB元器件。
3.  支持从原理图更新PCB文件。
4.  支持放置、移动、旋转PCB元器件。

除了添加PCB元器件相关的工具外，还对已有工具进行了增强。

1.  增强了原理图连线工具。对工具的功能和参数进行了优化调整，原来的连线工具在很多情况下会失败，目前大部分场景能够找到合适的连线路径。
2.  添加了大语言模型上下文空间管理功能。该功能会剔除过时的工具调用结果，节省上下文空间。当上下文超过压缩阈值时MCP会利用LLM来压缩历史聊天，尽量保留用户意图和设计方案。
3.  优化了插件界面显示，合并了对话和工具调用窗口。打开KiCad API的情况下支持通过IPC接口更新PCB编辑器显示。但由于接口的限制，暂时无法更新原理图编辑器显示，需要用户手动刷新。
4.  支持保存、恢复、重置当前会话。
5.  支持保存快照，方便用户回退修改。LLM有时会搞砸原理图连线，乱到需要重新连线。
6.  增加了测试用例，修复代码问题。

新的插件界面如下所示

![img](/images/kicad-plugin-pcb-interface-v2.png)

相关文章：

1.  [如何使用LLM直接修改KiCad原理图](https://paul356.github.io/2026/04/09/kicad-mcp.html)
2.  [基于KiCad的大语言模型插件(Agent)](https://paul356.github.io/2026/04/24/kicad-plugin.html)


## PCB相关功能介绍


### PCB功能设计

KiCad的PCB文件（.kicad\_pcb）和原理图文件一样，也是S表达式格式的纯文本文件，可以直接用代码读写。PCB相关功能的实现主要分两个部分。

第一个部分是封装的选型和同步。用户在原理图阶段为每个元件符号指定封装（Footprint属性），指定的封装名来自系统封装库。为了让LLM知道系统里有哪些封装可以用，插件会扫描系统的fp-lib-table，把所有.kicad\_mod文件的元数据写入一个SQLite索引库，供LLM搜索查询。封装选好并写入原理图后，再通过KiCad的IPC接口触发"Update PCB from Schematic"操作，把封装同步到PCB文件里。

第二个部分是PCB上的元件摆放。同步完成后，封装坐标、旋转角度等信息都存在.kicad\_pcb文件里，插件直接解析这个文件来读取和修改封装位置。修改完成后调用IPC接口通知KiCad重新加载PCB文件，PCB编辑器就会自动刷新显示。


### PCB相关工具

插件内嵌的MCP工具添加了下列工具以支持PCB相关的功能。工具列表如下。

| #  | 工具名                       | 作用                  |
|--- |---------------------------- |--------------------- |
| 1  | sync\_footprint\_index       | 后台构建/增量更新封装库索引 |
| 2  | get\_footprint\_sync\_status | 查询封装索引构建进度  |
| 3  | list\_footprint\_libraries   | 列出所有可用封装库    |
| 4  | search\_footprints           | 按名称/描述/标签搜索封装 |
| 5  | get\_footprint\_details      | 获取封装详情（焊盘、边界框等） |
| 6  | get\_board\_info             | 获取 PCB 板基本信息   |
| 7  | list\_footprints             | 列出板上已放置封装    |
| 8  | get\_footprint               | 获取单个封装详情      |
| 9  | list\_nets                   | 列出板上所有网络      |
| 10 | get\_ratsnest                | 获取未布线飞线（ratsnest） |
| 11 | get\_board\_outline          | 读取板框（Edge.Cuts）图形元素 |
| 12 | clear\_board\_outline        | 清除板框              |
| 13 | add\_board\_outline\_segment | 向板框添加直线段      |
| 14 | add\_board\_outline\_arc     | 向板框添加圆弧        |
| 15 | set\_board\_outline\_rect    | 一步设置矩形板框（支持圆角） |
| 16 | get\_footprint\_bbox         | 获取单个封装的 courtyard 边界框 |
| 17 | get\_board\_bounding\_box    | 获取全板封装的联合边界框 |
| 18 | align\_footprints            | 将一组封装对齐到同一坐标轴 |
| 19 | distribute\_footprints       | 沿坐标轴等间距分布封装 |
| 20 | move\_footprints\_by\_delta  | 将一组封装整体平移 (dx, dy) |
| 21 | find\_free\_pcb\_area        | 搜索板上不与已有封装重叠的空闲位置 |
| 22 | set\_footprint\_position     | 移动/旋转单个封装     |
| 23 | flip\_footprint              | 将封装在顶层/底层之间翻转 |
| 24 | set\_footprint\_property     | 设置封装属性字段      |
| 25 | update\_pcb\_from\_schematic | 触发"从原理图更新 PCB"操作 |
| 26 | reload\_kicad                | 在 KiCad 编辑器中重新加载文件 |


### 工具调用举例

下面用一个实际的例子介绍如何用这个插件来设计PCB。因为KiCad 10.0与中文输入法不太兼容，为了方便我输入的是英文提示词。也可以在文件编辑器中输入中文再粘贴到插件中，我因为懒就直接用了英文。

1.  我用KiCad AI Assistant插件画了一个由USB和电池组成的简单电源电路。
    -   ![img](/images/kicad-plugin-pcb-schematic.png)
2.  在KiCad AI Assistant中要求大语言模型帮我们搜索适合的元器件封装。
    -   ![img](/images/kicad-plugin-pcb-search-footprint.png)
3.  大语言模型搜索封装库，选好封装后把封装信息写入原理图。
    -   ![img](/images/kicad-plugin-pcb-set-footprint.png)
4.  待确定完所有元器件后让大语言模型触发基于原理图更新PCB的操作，KiCad会弹出"Update PCB from Schematic"对话框，用户点击确认后封装就出现在PCB上了。
    -   ![img](/images/kicad-plugin-pcb-update-pcb-from-schematic.png)
    -   ![img](/images/kicad-plugin-pcb-footprints-on-pcb.png)
5.  继续让大语言模型调整PCB元器件的位置，直到所有元器件都摆放到合适的位置。
    -   ![img](/images/kicad-plugin-pcb-arrange-prompt.png)
    -   ![img](/images/kicad-plugin-pcb-arrange-footprints.png)

目前KiCad AI Assistant还不支持PCB布线功，还需要用户自己布线，不久将会支持布线功能。


## 总结

本文介绍了最近给KiCad AI Assistant插件新增的PCB相关功能，包括封装库搜索、PCB查询和编辑以及元器件放置等工具。同时也对已有的连线工具和上下文管理做了改进，提升了连线稳定性，还添加了其他易用性相关的优化。目前还有一个小限制：原理图编辑器的界面需要用户手动刷新。

在开发这个插件工具的过程中，我发现做出一个MCP很容易，但做出一个好用的MCP还需要很多的打磨。希望KiCad AI Assistant能对大家有帮助，欢迎大家试用，反馈问题，提交PR。


## 参考索引

1.  kicad-mcp克隆 - <https://github.com/paul356/kicad-mcp>

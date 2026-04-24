---
layout: default
title: 基于KiCad的大语言模型插件(Agent)
tags: [LLM, MCP, KiCad]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 基于KiCad的大语言模型插件(Agent)


## 前言

这一篇是上一篇“如何使用LLM直接修改KiCad原理图”的延伸。我把前一篇水文转给一个朋友看，这个朋友建议我提升一下易用性。我觉得这个朋友讲的很有道理，所以调整了开发的计划，先开发一个KiCad插件，让用户可以直接在KiCad中使用这个Agent。这个KiCad插件集成了KiCad MCP，用户可以通过这个Agent来辅助设计电路和PCB。

相关文章：

1.  [如何使用LLM直接修改KiCad原理图](https://paul356.github.io/2026/04/09/kicad-mcp.html)


## 插件介绍


### 功能概述

该插件拥有一个图形界面，用户通过该界面使用自然语言来查看、修改当前项目中的电路原理图和PCB电路图。安装过程也比较简单。

![img](/images/kicad-plugin-interface.png)


### 安装方法

这里描述的安装方法针对KiCad 10.0的Linux发行版，其他平台和版本受限于工作量还未测试。安装该插件需要事先安装Python环境管理工具uv，安装过程中需要通过uv来创建Python虚拟环境供KiCad MCP使用。

由于该插件仍在开发过程中，当前安装过程中还需要下载代码包，后面会进一步优化安装方法。整个安装过程如下。

1.  从[kicad-mcp克隆](https://github.com/paul356/kicad-mcp)下载代码。
2.  进入代码目录执行 `make dist-plugin` 命令。
3.  命令成功后，将 `dist` 目录下的 `kicad_ai_assistant.zip` 文件解压到 `\~/.local/share/kicad/10.0/scripting/plugins` 目录下。
4.  进入plugins目录下的 `kicad_ai_assistant` 目录，执行脚本 `setup_plugin.sh <kicad-mcp项目路径>` 。

成功执行完这四步，插件就安装好了。


### 使用方法

接下来打开一个KiCad项目，从PCB Editor的Tools菜单打开Kicad AI Assistant插件。

![img](/images/kicad-plugin-open-plugin.png)

第一次使用该工具之前，需要先配置大模型。配置界面如下，用户需填入大模型服务节点（使用OpenAI协议）、API Key、模型名称等信息。

![img](/images/kicad-plugin-setting-interface.png)

配置好了大模型之后，就可以使用大语言模型来帮助我们分析、修改电路原理图和PCB了。目前插件支持如下的操作功能。

| #  | 工具名                            | 作用                  |
|--- |--------------------------------- |--------------------- |
| 1  | extract\_project\_netlist         | 从项目提取网络表      |
| 2  | extract\_schematic\_netlist       | 从单个 .kicad\_sch 提取网络表 |
| 3  | find\_component\_connections      | 查找指定元件的连接关系 |
| 4  | sync\_symbol\_index               | 同步/重建符号库索引   |
| 5  | get\_symbol\_sync\_status         | 查询符号索引状态      |
| 6  | search\_symbols                   | 搜索符号              |
| 7  | get\_symbol                       | 获取符号详情          |
| 8  | list\_symbol\_libraries           | 列出符号库            |
| 9  | get\_library\_symbols             | 列出库中的符号        |
| 10 | get\_symbol\_index\_stats         | 符号索引统计          |
| 11 | get\_symbol\_pins                 | 获取符号引脚信息      |
| 12 | add\_symbol\_to\_schematic        | 向原理图添加符号      |
| 13 | remove\_symbol\_from\_schematic   | 从原理图删除符号      |
| 14 | set\_component\_property          | 设置元件属性          |
| 15 | list\_component\_properties       | 列出元件属性          |
| 16 | delete\_component\_property       | 删除元件属性          |
| 17 | move\_component                   | 移动元件              |
| 18 | add\_label\_to\_schematic         | 添加标签（label）     |
| 19 | list\_labels\_in\_schematic       | 列出原理图标签        |
| 20 | delete\_label\_from\_schematic    | 删除标签              |
| 21 | add\_wire\_to\_schematic          | 添加导线              |
| 22 | connect\_pins\_with\_wire         | 用导线连接两个引脚    |
| 23 | delete\_wire\_from\_schematic     | 删除导线              |
| 24 | add\_junction\_to\_schematic      | 添加节点（junction）  |
| 25 | list\_junctions\_in\_schematic    | 列出节点              |
| 26 | delete\_junction\_from\_schematic | 删除节点              |
| 27 | sync\_footprint\_index            | 同步封装库索引        |
| 28 | get\_footprint\_sync\_status      | 封装索引状态          |
| 29 | list\_footprint\_libraries        | 列出封装库            |
| 30 | search\_footprints                | 搜索封装              |
| 31 | get\_footprint\_details           | 获取封装详情          |
| 32 | get\_board\_info                  | 获取 PCB 板信息       |
| 33 | list\_footprints                  | 列出板上封装          |
| 34 | get\_footprint                    | 获取单个封装信息      |
| 35 | list\_nets                        | 列出网络              |
| 36 | get\_ratsnest                     | 获取飞线（ratsnest）  |
| 37 | set\_footprint\_position          | 设置封装位置          |
| 38 | flip\_footprint                   | 翻面（顶/底层）       |
| 39 | set\_footprint\_property          | 设置封装属性          |

大语言模型会根据用户请求和工具描述调用合适的工具，不过由于MCP工具设计和大语言模型能力的限制，建议大家先从简单一点的任务开始尝试。


## 总结

本文介绍了我最近伙同AI一起开发的一个KiCad插件，欢迎大家试用。该插件还在持续开发过程中，欢迎大家提建议，也欢迎有兴趣的朋友一起优化这个工具。


## 参考索引

1.  kicad-mcp克隆 - <https://github.com/paul356/kicad-mcp>

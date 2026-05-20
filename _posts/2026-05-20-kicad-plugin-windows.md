---
layout: default
title: 基于KiCad的大语言模型插件(Agent) - Windows平台安装指南
tags: [LLM, MCP, KiCad, Windows]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 基于KiCad的大语言模型插件(Agent) - Windows平台安装指南


## 前言

前面几篇文章介绍了 KiCad AI Assistant 插件的功能，但安装说明只覆盖了 Linux 平台。这篇文章介绍如何在 Windows 11 上安装并使用该插件，测试使用的 KiCad 版本为 10.0。

相关文章：

1.  [如何使用LLM直接修改KiCad原理图](https://paul356.github.io/2026/04/09/kicad-mcp.html)
2.  [基于KiCad的大语言模型插件(Agent)](https://paul356.github.io/2026/04/24/kicad-plugin.html)
3.  [基于KiCad的大语言模型插件(Agent) - 支持编辑PCB](https://paul356.github.io/2026/05/08/kicad-plugin-pcb.html)
4.  [基于KiCad的大语言模型插件(Agent) - 支持自动布线](https://paul356.github.io/2026/05/11/kicad-plugin-autoroute.html)


## 安装过程


### 第一步：安装 uv

插件依赖 [uv](https://github.com/astral-sh/uv) 来管理 Python 虚拟环境。在 Windows 上推荐使用 PowerShell 官方安装脚本进行安装：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

![img](/images/kicad-plugin-install-uv-win.png)

安装完成后，关闭并重新打开 PowerShell，执行下面的命令验证安装是否成功：

```powershell
uv --version
```


### 第二步：下载代码包和插件包

在 [kicad-mcp Releases](https://github.com/paul356/kicad-mcp/releases) 页面下载以下两个文件：

-   `kicad_ai_assistant.zip` — KiCad 插件包
-   `Source code (zip)` — kicad-mcp 源码包

将源码 ZIP 解压，把解压后的文件夹重命名为 `kicad-mcp` 并放到一个固定位置，例如 `C:\kicad-mcp` 。

将 `kicad_ai_assistant.zip` 解压到 KiCad 的插件目录 `%USERPROFILE%\Documents\KiCad\10.0\scripting\plugins` 下。

解压后目录结构如下：

```
%USERPROFILE%\Documents\KiCad\10.0\scripting\plugins\
└── kicad_ai_assistant\
    ├── __init__.py
    ├── setup_plugin.bat
    ├── setup_plugin.ps1
    └── ...
```


### 第三步：运行安装脚本

打开 PowerShell，进入刚才解压好的插件目录，执行安装脚本，并将 kicad-mcp 源码目录作为参数传入：

```powershell
cd "$env:USERPROFILE\Documents\KiCad\10.0\scripting\plugins\kicad_ai_assistant"
.\setup_plugin.bat C:\kicad-mcp
```

脚本会自动完成以下三件事：

1.  在插件目录下创建 Python 虚拟环境（ `.venv` ）；
2.  将 kicad-mcp 以可编辑模式安装到虚拟环境中；
3.  从 GitHub 下载 freerouting JAR 文件（用于自动布线功能）。

![img](/images/kicad-plugin-config-plugin-win.png)


## 配置插件

安装完成后，打开 KiCad，在 PCB 编辑器的 **Tools** 菜单中找到并启动 **Kicad AI Assistant** 插件。

![img](/images/kicad-plugin-open-plugin-win.png)

初次使用需要配置大模型参数。点击插件界面中的 **Options → Settings** 打开设置对话框，填写以下信息：

-   \*LLM Provider\*：选择 `openai` 、 `anthropic` 或 `custom`
-   \*API Key\*：对应服务的 API Key
-   \*Model\*：模型名称，例如 `deepseek-v4-pro` 或其他兼容模型
-   \*Custom endpoint URL\*：自定义服务地址（使用兼容 OpenAI 协议的第三方节点时填写）

![img](/images/kicad-plugin-setting-interface-win.png)

设置对话框还提供了上下文窗口相关参数，可根据所使用模型的规格进行调整。


## 使用方法

配置完成后，在插件的聊天窗口中用自然语言描述需求，AI 会自动调用相应的工具来分析或修改原理图与 PCB。建议从简单的查询类任务开始熟悉插件的使用方式，例如让 AI 列出当前原理图中的元件或查询某个网络的连接关系。

目前插件支持的工具与 Linux 版本相同，详见[基于KiCad的大语言模型插件(Agent)](https://paul356.github.io/2026/04/24/kicad-plugin.html)一文中的工具列表。


## 总结

本文介绍了如何在 Windows 11 + KiCad 10.0 平台上安装和配置 KiCad AI Assistant 插件，主要步骤为：安装 uv、从 GitHub Releases 下载并解压两个压缩包、执行 `setup_plugin.bat` 完成环境配置。如有问题，欢迎在 [kicad-mcp](https://github.com/paul356/kicad-mcp) 项目中提 Issue 反馈。


## 参考索引

1.  kicad-mcp 项目 - <https://github.com/paul356/kicad-mcp>
2.  kicad-mcp Releases - <https://github.com/paul356/kicad-mcp/releases>
3.  uv - <https://github.com/astral-sh/uv>
4.  uv 安装文档 - <https://docs.astral.sh/uv/getting-started/installation/>

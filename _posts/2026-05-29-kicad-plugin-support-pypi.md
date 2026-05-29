---
layout: default
title: 基于KiCad的大语言模型插件(Agent) — 简化安装方法
tags: [LLM, MCP, KiCad, pypi, pip]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 基于KiCad的大语言模型插件(Agent) — 简化安装方法


## 前言

之前安装KiCad AI Assistant需要从项目 Releases 页面下载源码包，手动解压并传入路径参数，流程较为繁琐。为了方便用户安装和升级，我对项目做了以下改进：

1.  项目迁移到独立代码仓库 [KiCad-AI-Assistant（见参考链接）](https://github.com/paul356/KiCad-AI-Assistant)，支持创建问题单。
2.  MCP Server 已发布到 [PyPI](https://pypi.org/project/kcaa/)，安装脚本会自动从 PyPI 安装 `kcaa` 包，\*无需再下载源码\*。
3.  安装脚本可自动检测 KiCad 版本和路径，无需手动传入参数。

同时，新版本修复了多个 Windows 平台的兼容性问题，大概率解决得差不多了，不过也不排除有漏网之鱼。

本系列相关文章：

1.  [如何使用LLM直接修改KiCad原理图](https://paul356.github.io/2026/04/09/kicad-mcp.html)
2.  [基于KiCad的大语言模型插件(Agent)](https://paul356.github.io/2026/04/24/kicad-plugin.html)
3.  [基于KiCad的大语言模型插件(Agent) - 支持编辑PCB](https://paul356.github.io/2026/05/08/kicad-plugin-pcb.html)
4.  [基于KiCad的大语言模型插件(Agent) - 支持自动布线](https://paul356.github.io/2026/05/11/kicad-plugin-autoroute.html)
5.  [基于KiCad的大语言模型插件(Agent) - Windows平台安装指南](https://paul356.github.io/2026/05/20/kicad-plugin-windows.html)
6.  [基于KiCad的大语言模型插件(Agent) — 解决Windows版本卡死问题](https://paul356.github.io/2026/05/26/kicad-plugin-hang-issue.html)


## 简化安装步骤


### 第一步：安装 uv

插件依赖 [uv](https://github.com/astral-sh/uv) 管理 Python 虚拟环境。

Linux：

```bash
curl -Lsf https://astral.sh/uv/install.sh | sh
```

Windows（PowerShell）：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

安装完成后重新打开终端，验证安装：

```bash
uv --version
```


### 第二步：下载并解压插件包

从 [Releases 页面](https://github.com/paul356/KiCad-AI-Assistant/releases) 下载 `kicad-ai-assistant.zip` ，解压到 KiCad 插件目录：

-   Linux： `/.local/share/kicad/10.0/scripting/plugins`
-   Windows： `%USERPROFILE%\Documents\KiCad\10.0\scripting\plugins`

解压后目录结构如下：

```
kicad_ai_assistant/
├── __init__.py
├── setup_plugin.sh     # Linux
├── setup_plugin.bat    # Windows
└── ...
```


### 第三步：运行安装脚本

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

安装脚本会自动完成以下操作：

1.  创建 Python 虚拟环境（ `.venv` ）；
2.  从 [PyPI](https://pypi.org/project/kcaa/) 安装 `kcaa` 包；
3.  自动检测 KiCad 版本和安装路径，生成 `.env` 配置文件；
4.  下载 freerouting JAR 文件（用于自动布线功能）。

与旧版不同，\*无需下载源码包\*，也\*无需手动传入路径参数\*。如果 KiCad 安装在非默认路径（如 Windows 上的 `D:\KiCad` ），脚本会提示手动输入正确的路径。


### Windows 兼容性改进

新版本针对 Windows 平台做了以下兼容性修复：

1.  \*默认编码问题\*：所有文件读写操作统一使用 UTF-8 编码，避免 Windows 默认 GBK 编码导致的乱码和解析错误。
2.  \*文件系统分隔符问题\*：统一使用正斜杠（ `/` ）处理路径，避免反斜杠（ `\` ）在 Python 字符串和配置文件中被转义。
3.  \*KiCad 安装目录问题\*：安装脚本自动检测 KiCad 安装路径，并允许用户在非默认路径下手动指定，不再硬编码单一路径。

这些修复让 Windows 用户可以获得与 Linux 一致的使用体验。


## 独立 MCP 服务器

除了插件模式， `kcaa` 也可以作为独立 MCP 服务器运行，与其他 MCP 客户端（如 Claude Desktop、Cursor）集成。首先安装 `kcaa` ：

```bash
pip install kcaa
```

在工作目录创建 `.env` 文件：

```
KICAD_SEARCH_PATHS=/home/user/pcb
KICAD_APP_PATH=/usr/share/kicad
KICAD_VERSION=10.0
KICAD_CONFIG_DIR=~/.config/kicad/10.0
KICAD_3RD_PARTY=~/.local/share/kicad/10.0/3rdparty
MCP_TRANSPORT=streamable-http
```

然后启动服务器：

```bash
kcaa
```


## 总结

本文介绍了 KiCad AI Assistant 全新的简化安装方式。原代码仓库将不再维护，将转为只读仓库。大家可以到新代码仓库的 Release 页面获取安装包。最后再附上技术交流群二维码，欢迎进群反馈问题和交流想法。

![img](/images/kicad-plugin-wexin-group-chat.jpg)


## 参考链接

1.  KiCad-AI-Assistant 项目 - <https://github.com/paul356/KiCad-AI-Assistant>
2.  kcaa PyPI 包 - <https://pypi.org/project/kcaa/>
3.  Releases 页面 - <https://github.com/paul356/KiCad-AI-Assistant/releases>
4.  uv - <https://github.com/astral-sh/uv>

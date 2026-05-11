---
layout: default
title: 基于KiCad的大语言模型插件(Agent) - 支持自动布线
tags: [LLM, MCP, KiCad, PCB, freerouting]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 基于KiCad的大语言模型插件(Agent) - 支持自动布线


## 前言

前面介绍了如何通过KiCad AI Assistant修改PCB，当时还未想好怎么支持自动布线。没想到今天我遇到了freerouting这个工具，该工具支持KiCad等多种PCB编辑软件，使用起来也很方便。因此通过集成freerouting工具，我们就可以补上自动布线功能，这篇文章就来介绍KiCad AI Assistant是如何实现这一功能的。

相关文章：

1.  [如何使用LLM直接修改KiCad原理图](https://paul356.github.io/2026/04/09/kicad-mcp.html)
2.  [基于KiCad的大语言模型插件(Agent)](https://paul356.github.io/2026/04/24/kicad-plugin.html)
3.  [基于KiCad的大语言模型插件(Agent) - 支持编辑PCB](https://paul356.github.io/2026/05/08/kicad-plugin-pcb.html)


## 自动布线功能


### 集成freerouting

[freerouting](https://github.com/freerouting/freerouting) 是一个成熟的开源自动布线工具，使用 Specctra DSN/SES 格式与 KiCad 交换数据。KiCad 可以将 PCB 导出为 DSN 文件，freerouting 读取 DSN 后完成布线并输出 SES 文件，KiCad 再将 SES 导入回 PCB。

第一步，用 pcbnew Python API 将当前 PCB 导出为 DSN 文件：

```python
import pcbnew
board = pcbnew.GetBoard()
pcbnew.ExportSpecctraDSN(board, "/tmp/kicad_autoroute_xxx/board.dsn")
```

第二步，调用 freerouting JAR 进行自动布线：

```bash=
java -jar freerouting-2.2.3.jar \
    -de /tmp/kicad_autoroute_xxx/board.dsn \
    -do /tmp/kicad_autoroute_xxx/board.ses \
    --gui.enabled=false \
    -mp 50
```

第三步，布线完成后将 SES 文件导回 PCB 并保存：

```python
import pcbnew
board = pcbnew.GetBoard()
pcbnew.ImportSpecctraSES(board, "/tmp/kicad_autoroute_xxx/board.ses")
```

freerouting 本身是一个 Java 程序，以 JAR 包形式发布。为了方便部署，插件目录里提供了 `setup_plugin.sh` 脚本，会在安装插件依赖的同时自动从 GitHub Release 下载指定版本的 freerouting JAR（当前版本为 2.2.3）并放置到插件目录下。

```bash
# 在插件目录中执行，一键完成依赖安装和 JAR 下载
cd ~/.local/share/kicad/10.0/scripting/plugins/kicad_ai_assistant
./setup_plugin.sh ~/code/kicad-mcp
```

脚本会帮我们创建python虚拟环境和下载freerouting.jar文件。


### 如何使用自动布线

在完成封装摆放之后，就可以通过菜单启动自动布线。步骤如下。

1.  确保 PCB 编辑器中已打开需要布线的 .kicad\_pcb 文件，且封装已摆放到位。
    -   ![img](/images/kicad-plugin-autoroute-before-route.png)
2.  在 KiCad AI Assistant 插件窗口的菜单栏中选择 \*Tools → Auto Route…\*。
    -   ![img](/images/kicad-plugin-autoroute-menu.png)
3.  插件会弹出网络选择对话框，列出板上所有网络。可以勾选不希望 freerouting 自动布线的网络（例如 GND、电源网络等，通常这些网络需要手动铺铜处理）。如果不选择任何网络，则对全部网络布线。
    -   ![img](/images/kicad-plugin-autoroute-skip-nets.png)
4.  点击确认后，插件先导出 DSN 文件，同时为当前 PCB 保存一个快照（方便布线效果不理想时回退）。
5.  freerouting 在后台运行（最多 50 次优化遍历），界面状态栏会显示"⏳ Auto-routing in progress…"。
6.  布线完成后，插件自动将结果导入 PCB，保存文件并刷新编辑器视图，状态栏变为"✅ Backend ready"。
    -   ![img](/images/kicad-plugin-autoroute-result.png)


## 总结

本文介绍了 KiCad AI Assistant 插件新增的自动布线功能。通过集成开源工具 freerouting，结合之前已有的封装搜索、元器件摆放等功能，用户现在可以在 KiCad AI Assistant 的辅助下完成从原理图到布线完成的大部分 PCB 设计流程。

插件代码可以从文末参考索引中的 `kicad-mcp` 仓库获取，欢迎大家试用，反馈问题，提交 PR。


## 参考索引

1.  kicad-mcp - <https://github.com/paul356/kicad-mcp>
2.  freerouting - <https://github.com/freerouting/freerouting>

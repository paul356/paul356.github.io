---
layout: post
title: 键盘计划
excerpt: 键盘计划
---


# 设计要素


## 硬件

-   芯片
    -   主控: esp-c2
    -   供电
    -   充电: x
-   显示: OLED
-   电池: x
-   蜂鸣器
-   pcb
    -   印制
    -   贴片
-   轴体
-   壳体
    -   3d建模
    -   仿真
    -   模具
-   键帽
-   定位板
-   静音缓冲


## 软件


### 基本功能

-   电源开关（暂无）
-   状态显示
    -   [X] 支持屏幕显示
    -   [X] 显示运行状态
    -   提示状态变化
        -   [ ] 运行 -> 待机
-   连接管理
    -   USB连接相关
        -   [X] USB Device驱动
        -   [X] 选择是否通过USB发送按键
    -   蓝牙相关
        -   [X] 蓝牙Server驱动
        -   [ ] 打开蓝牙
        -   [ ] 关闭蓝牙
        -   [ ] 重置连接
        -   [ ] 支持多连接
        -   [X] 蓝牙状态显示
-   键盘输入
    -   [X] 移植QMK键盘软件
-   在线升级
    -   [X] HTTP OTA升级
-   充电
    -   [ ] 显示电量
    -   [ ] 展示充电状态
-   节能
    -   [ ] 支持USB Suspend
    -   [ ] 支持自动进入IDLE节能


### 高级功能

-   布局相关
    -   [X] 布局展示
    -   [X] 布局制定
-   定制宏
    -   [ ] 支持宏录制
-   统计功能
    -   [ ] 支持输入直方图
-   键记录
-   智能纠错


# 开发计划

计划时间安排:

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">时间阶段</th>
<th scope="col" class="org-left">内容</th>
<th scope="col" class="org-left">备注</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">2023/02/25 ~ 2023/02/28</td>
<td class="org-left">蓝牙驱动、连接管理</td>
<td class="org-left">解决蓝牙连接问题，支持开关蓝牙、usb连接等</td>
</tr>


<tr>
<td class="org-left">2023/03/01 ~ 2023/03/07</td>
<td class="org-left">布局展示、键码定制</td>
<td class="org-left">支持键布局定制</td>
</tr>


<tr>
<td class="org-left">2023/03/08 ~ 2023/03/15</td>
<td class="org-left">显示驱动、蜂鸣器</td>
<td class="org-left">尝试增加外设</td>
</tr>


<tr>
<td class="org-left">2023/03/15 ~ 2023/03/31</td>
<td class="org-left">PCB设计、外壳设计</td>
<td class="org-left">设计对用户友好的硬件布局，以及外壳</td>
</tr>


<tr>
<td class="org-left">2023/04/01 ~ 2023/04/30</td>
<td class="org-left">外壳试制、PCB生产、贴片</td>
<td class="org-left">PCB贴片</td>
</tr>


<tr>
<td class="org-left">2023/05/08 ~ 2023/05/15</td>
<td class="org-left">美化界面</td>
<td class="org-left">使用numl.design美化界面</td>
</tr>
</tbody>
</table>


# 问题列表

现有问题列表:

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-right" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-right">问题ID</th>
<th scope="col" class="org-left">描述</th>
<th scope="col" class="org-left">问题原因</th>
<th scope="col" class="org-left">解决方法</th>
<th scope="col" class="org-left">计划解决时间</th>
<th scope="col" class="org-left">当前状态</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-right">001</td>
<td class="org-left">蓝牙无法连接</td>
<td class="org-left">battery_set导致挂死，无法通过indication发送按键</td>
<td class="org-left">去掉battery_set操作</td>
<td class="org-left">2023/02/28</td>
<td class="org-left">已解决</td>
</tr>


<tr>
<td class="org-right">002</td>
<td class="org-left">蓝压断开后无法恢复连接</td>
<td class="org-left">未知</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">2023/03/07</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-right">003</td>
<td class="org-left">usb在电脑待机后没有响应</td>
<td class="org-left">未知</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">2023/03/07</td>
<td class="org-left">&#xa0;</td>
</tr>
</tbody>
</table>


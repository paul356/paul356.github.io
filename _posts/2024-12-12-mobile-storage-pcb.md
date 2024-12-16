---
layout: default
title: 一种基于ESP32丰富连接能力的移动存储设备 -- 电路设计
tags: [Rust on ESP, ESP32, PCB]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 一种基于ESP32丰富连接能力的移动存储设备 &#x2013; 电路设计


## 前言

之前我构想过一种[基于ESP32的移动存储设备](https://paul356.github.io/2024/10/31/mobile-storage.html)，现在来尝试做一个样机，第一步我觉得先要设计一个专门的PCB。之前我使用了一款ESP32S3开发板来测试如何运行WebDAV，但由于开发板的闪存空间只有几个兆的容量。所以我觉得需要设计一个新的PCB电路，以支持更大的存储空间，我选择的方式是为ESP32S3增加TF卡。我目前还是一个设计PCB的业余选手，设计过程中主要参考了nanoESP32-S3和esp32-thing两块开发板，所以PCB中非常可能有不少问题，还欢迎大家指正。


## 电路设计


### 准备工作

设计PCB需要使用设计工具，我目前使用的工具是[KiCad](https://www.kicad.org/)。这是一款开源软件，支持绘制电路原理图和PCB电路，在开源硬件社区比较出名。KiCad8还支持插件工具，可以一键生成提交给PCB厂家的gerber文件，非常方便的。

下面是KiCad8的主界面，在这里创建项目。我创建了一个叫做esp32s3-storage的项目，在左边文件列表可以看到有esp32s3-storage.kicad\_sch和esp32s3-storage.kicad\_pcb，这两个文件分别代表了电路原理图和PCB电路图。 ![img](/images/kicad8-main-window.jpg)


### 电路原理图

目前电路原理图还比较简单，分为5个部分，分别是电源、USB接口、MicroSD卡、ESP32S3芯片、各种连接器。将各个器件连接好后，实现了一个支持USB或电池供电的有SPI显示接口和MicroSD卡槽的ESP32S3小系统。最后电路图看起来像这个样子。 ![img](/images/esp32-storage-sch.jpg)


### PCB电路图

原理图做好后，再确定好元器件对应的元件封装就可以绘制PCB电路图了。下图就是最后绘制好的PCB电路，图片中红色和蓝色折线是连接电子元器件的导线，带有标识的是元器件的摆放位置。右上边U1就是ESP32S3，右下边J1是USB-C接口，左上角J3是MicroSD卡的位置，左下角J2是5V电池输入接口。右部边缘有两排接口，长的是还未分配的GPIO口，短一点的用于接SPI接口的显示模块。另外还有两个短一点的GPIO接口分布在板子中间。剩下的板面都填充成GND区域，希望能保证信号的稳定。 ![img](/images/esp32-storage-pcb.jpg)

PCB电路图做好后，通过插件Frabrication Toolkit（一样通过Plugin and Content Manager安装，直接搜索可以安装）生成gerber文件，提交给PCB厂家就可以制作PCB了。 ![img](/images/export-production-files.jpg)


## 后记

本文中的电路原理图和PCB电路文件可以从[这里](https://github.com/paul356/esp32s3-storage)下载。PCB厂家的效率很高，我的短文还没有写完，做好的PCB就已经到了。成品长这个样子，除了黑色的油漆层，是不是KiCad8中看起来差不多。下一步就是焊板子，做测试，有进展了再给大家汇报。 ![img](/images/esp32-storage-real-pcb.jpg)

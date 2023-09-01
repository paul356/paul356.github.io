---
layout: default
title: 创客利器之ESP32（二）
tags: [esp32, MCU, micropython]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 创客利器之ESP32（二）

上一篇文章“创客利器之ESP32（一）”介绍了什么ESP32，以及通过Arduino来对ESP32进行编程。今天我们再了解一个更简单快捷地对ESP32编程的方法：MicroPython环境。先看一下MicroPython的界面。

![img](/images/micropython_webrepl.jpg)

界面非常朴实无华，通过这个界面我们可以便捷地控制ESP32。左上角的地址"ws://192.168.4.1:8266"代表ESP32上运行的webrepl服务的地址。中间是Python程序的交互窗口，可以在这里输入程序。右边的"Send a file"和"Get a file"可以上传和下载文件。


## MicroPython环境介绍

MicroPython其实就是在嵌入式芯片上运行的Python环境，可以通过Python控制嵌入式系统。Python由于其简单的语法，和交互性的解释环境，可以边实验边开发程序。是不是想动手试一试了呢，不过我们得先给ESP32安装MicroPython程序。


## 安装MicroPython环境

安装MicroPython环境也很简单，通过Python3的扩展库管理工具pip3安装工具esptool，再通过esptool将MicroPython环境写到ESP32上就可以了。安装Python3的步骤，可以搜索网络上的教程。

1.  安装esptool工具
2.  安装MicroPython环境
3.  对MicroPython环境进行配置


### 第一步 - 安装esptool工具

使用pip3安装方法很简单，在命令行里输入下列命令。

    pip3 install esptool

国内访问pip3服务器有时候不是太稳当，如果一直安装失败，可以设置pip3的镜像。


### 第二步 - 安装MicroPython环境

先从这里[micropython.org esp32](https://micropython.org/download/esp32/)下载安装文件。因为发布版本的wifi功能有些问题，我使用了Nightly builds的bin文件（esp32-20230323-unstable-v1.19.1-992-g38e7b842c.bin）。安装用到两条命令。第一个命令擦除ESP32 flash上原有的程序。注意这里的/dev/cu.usbserial-0001是Mac下的串口设备，Linux系统下可能是/dev/ttyUSB0，windows系统下可能是COM1。需要先找到ESP32对应的串口设备，并替换这里的/dev/cu.usbserial-0001。

    esptool.py --chip esp32 --port /dev/cu.usbserial-0001 erase_flash

成功的输出

    esptool.py v3.3.2
    Serial port /dev/cu.usbserial-0001
    Connecting.................
    Chip is ESP32-D0WDQ6 (revision 1)
    Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
    Crystal is 40MHz
    MAC: 78:21:84:7f:ef:a4
    Uploading stub...
    Running stub...
    Stub running...
    Erasing flash (this may take a while)...
    Chip erase completed successfully in 6.1s
    Hard resetting via RTS pin...

第个命令用esptool.py工具将bin文件写到ESP32上。

    esptool.py --chip esp32 --port /dev/cu.usbserial-0001 --baud 460800 write_flash -z 0x1000 ~/Downloads/esp32-20230323-unstable-v1.19.1-992-g38e7b842c.bin

成功的输入

    esptool.py v3.3.2
    Serial port /dev/cu.usbserial-0001
    Connecting............
    Chip is ESP32-D0WDQ6 (revision 1)
    Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
    Crystal is 40MHz
    MAC: 78:21:84:7f:ef:a4
    Uploading stub...
    Running stub...
    Stub running...
    Changing baud rate to 460800
    Changed.
    Configuring flash size...
    Flash will be erased from 0x00001000 to 0x0017ffff...
    Compressed 1566496 bytes to 1034680...
    Wrote 1566496 bytes (1034680 compressed) at 0x00001000 in 24.9 seconds (effective 503.3 kbit/s)...
    Hash of data verified.
    
    Leaving...
    Hard resetting via RTS pin...

第一步可能会出现无法连接ESP32的错误，出现 `Connecting............` ，然后工具报无法进入下载模式的错误。

    esptool.py v3.3.2
    Serial port /dev/cu.usbserial-0001
    Connecting......................................
    
    A fatal error occurred: Failed to connect to ESP32: Wrong boot mode detected (0x13)! The chip needs to be in download mode.

如果出现这种情形，需要手动帮助ESP32进入可编程模式。具体做法是：在出现 `Connecting...` 时，先按住"Boot"按钮，再轻按一下"EN"按钮，再释放"Boot"按钮。写好MicroPython后，我们就有一个运行了MicroPython的ESP32设备了。


### 第三步 - 配置MicroPython环境

其实这时候我们已经可以开始使用MicroPython，不过这时只能通过串口访问MicroPython环境。

    screen /dev/cu.usbserial-0001 115200

然后进入一个交互窗口，在 `>>>` 处可以输入Python程序，就像使用PC上的python3程序一样。

    ets J�a�"�etfrv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
    mode:DIO, clock div:2
    load:0x3fff0030,len:4656
    load:0x40078000,len:13284
    ho 0 tail 12 room 4
    load:0x40080400,len:3712
    entry 0x4008064c
    Started webrepl in normal mode
    MicroPython v1.19.1-992-g38e7b842c on 2023-03-23; ESP32 module with ESP32
    Type "help()" for more information.
    >>> 

但是MicroPython还支持webrepl可以通过


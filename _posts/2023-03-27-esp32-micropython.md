---
layout: default
title: 创客利器之ESP32（二）
tags: [esp32, MCU, micropython]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 创客利器之ESP32（二）


## 前言

上一篇小文“创客利器之ESP32（一）”介绍了什么是ESP32，以及如何通过Arduino来对ESP32进行编程。今天我们再了解一个更简单快捷对ESP32编程的方法：MicroPython。先看一下MicroPython的界面。
![img](/images/micropython_webrepl.jpg)

界面非常简单，分为几个区域。左上角的地址"ws://192.168.4.1:8266"代表webrepl服务的地址。中间是Python程序的交互窗口，可以在这里输入程序。右边的"Send a file"和"Get a file"可以上传和下载文件，可以让我们上传写好的Python程序文件。


## MicroPython环境介绍

MicroPython其实就是在嵌入式芯片上运行的Python环境，可以通过Python控制嵌入式系统。Python由于简单的语法和交互性的解释环境，可以边实验边开发程序。是不是想动手试一试，不过我们得先给ESP32安装MicroPython程序。


## 安装MicroPython环境

安装MicroPython环境，是将MicroPython的固件刷到ESP32上。整个流程需要在电脑上先安装Python3，安装步骤可以参考网络上的教程。安装配置流程如下：

1.  安装esptool工具
2.  安装MicroPython环境
3.  对MicroPython环境进行配置


### 第一步 - 安装esptool工具

esptool可以通过pip3（python3的扩展包管理工具）安装，在Python命令行里输入下列命令。

    pip3 install esptool

国内使用pip3可能会遇到网络连接不稳定的情况。如果一直遇到安装失败，可以为pip3设置国内镜像源。


### 第二步 - 安装MicroPython环境

先从这里[micropython.org esp32](https://micropython.org/download/esp32/)下载安装文件。因为发布版本的WiFi功能有些问题，我使用了Nightly builds的bin文件（esp32-20230323-unstable-v1.19.1-992-g38e7b842c.bin）。安装共用到两条命令。

第一条命令擦除ESP32 flash上原有的程序。

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

第二条命令用esptool.py工具将bin文件写到ESP32上。

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

注意这两个命令里的/dev/cu.usbserial-0001是我的Mac下的串口设备名称，在Linux系统下可能是/dev/ttyUSB0，在Windows系统下可能是COM1。因为在不同电脑上串口设备名称会有些许差别，需要先找到ESP32对应的串口设备名称，替换这里的/dev/cu.usbserial-0001。另外还可能会出现无法连接ESP32的错误，出现 `Connecting............` ，然后工具打印无法进入下载模式的错误。

    esptool.py v3.3.2
    Serial port /dev/cu.usbserial-0001
    Connecting......................................
    
    A fatal error occurred: Failed to connect to ESP32: Wrong boot mode detected (0x13)! The chip needs to be in download mode.

如果出现这种情形，需要手动帮助ESP32进入可编程模式。具体做法是：在出现 `Connecting...` 时，先按住"Boot"按钮，再轻按一下"EN"按钮，再释放"Boot"按钮。顺利的话，执行完两条命令后，我们就有一个运行了MicroPython的ESP32设备了。


### 第三步 - 配置MicroPython环境

其实这时我们已经可以开始使用MicroPython了，不过只能通过串口访问MicroPython环境。Mac和Linux可以通过screen通过串口连接ESP32。Windows环境上可以使用putty程序通过串口访问环境。不过需要调整串口设置，波特率设置成115200，Parity和FlowControl都设置成None，默认的串口参数连接MicroPython有问题。这里使用screen作为示例：

    screen /dev/cu.usbserial-0001 115200

然后进入一个文本交互窗口，在 `>>>` 处输入Python程序，类似PC上的python3程序。

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

MicroPython还支持通过浏览器访问，不过需要做一些设置。先把ESP32设置成WiFi的热点，这样我们可以通过WiFi连接到ESP32。使用的代码如下：

    import network
    ap = network.WLAN(network.AP_IF)
    ap.config(ssid="ESP32-AP")
    ap.active(True)

这段代码的输入过程如下：

    >>> import network
    >>> ap = network.WLAN(network.AP_IF)
    >>> ap.config(ssid="ESP32-AP")       // WiFi热点的名字
    >>> ap.active(True)                  // 打开WiFi热点
    True
    >>> 

打开WiFi热点后，我们可以在无线网络管理里面找到"ESP32-AP"。可以尝试连接这个热点，看能否连接成功。

设置了WiFi之后，再配置webrepl服务，通过下面代码。

    import webrepl_setup

输入这行代码后，MicroPython回提示为webrepl设置4-9位字符的访问密码：

    >>> import webrepl_setup
    WebREPL daemon auto-start status: enabled
    
    Would you like to (E)nable or (D)isable it running on boot?
    (Empty line to quit)
    > E
    Would you like to change WebREPL password? (y/n) y
    New password (4-9 chars): xxxxxx
    Confirm password: xxxxxx
    No further action required

最后从[micropython/webrepl项目](https://github.com/micropython/webrepl/archive/refs/heads/master.zip)下载web客户端。解压zip文件后，使用浏览器打开webrepl.html。输入"ws://192.168.4.1:8266"，点击Connect，再输入密码。
![img](/images/webrepl_enlarge.jpg)
现在就可以输入Python程序了，至此MicroPython的安装和配置就完成了。


## 后记

上面我们完成了MicroPython的安装和配置，后面我们就可以使用MicroPython来对ESP32编程。可以尝试用MicroPython做一下算术运算，当然MicroPython可以做更多有意思的事情。下一篇文章我们会用MicroPython写一点有意思的程序。退出MicroPython之后不要忘了将电脑的WiFi从ESP32的热点断开，连接原来的Wi-Fi热点。


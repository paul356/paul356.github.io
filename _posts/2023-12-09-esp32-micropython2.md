---
layout: default
title: 创客利器之ESP32（三）- 使用MicroPython自动连接WiFi
tags: [esp32, MCU, micropython]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 创客利器之ESP32 (三) - 使用MicroPython自动连接WiFi


## 前言

上一篇文章“创客利器之ESP32（二）”介绍了怎么给ESP32安装MicroPython固件，以及如何通过webrepl访问ESP32上的Python解释器。接下来我们就用MicroPython来实现一个智能家居终端，可以通过手机控制电器的开关。不过这里面需要使用到许多组件，配合关系比较复杂，为了保证大家更容易理解，我会分成几步来讲解。这一篇主要讲解怎么用MicroPython来快速连接我们的家庭WiFi。


## 如何连接家庭WiFi

将ESP32通过MicroUSB数据线连接到我们的电脑。我使用的是linux环境，USB设备类似/dev/ttyUSBx，可以执行 `ls /dev/ttyUSB*` 查看一下。我这里使用 `screen /dev/ttyUSB0 115200` 连接MicroPython环境。注意在Mac下USB设备的格式是/dev/cu.usbserial-x，而Windows下是COMx，需要我们先找到设备名称。当前我们通过串口工具连上MicroPython环境后，输入下面的Python代码，就可以连接我们家里的热点。

```python
import network
sta_if = network.WLAN(network.STA_IF);
sta_if.active(True)
sta_if.scan()                             # 扫描热点
sta_if.connect("<AP_name>", "<password>") # 连接热点
sta_if.isconnected()                      # 检查是否连接上热点
print('Network Configuration:', sta_if.ifconfig()) # 查看IP信息
```

执行结果如下，注意执行代码里的真实WiFi名和密码被我隐去了。另外我还打开了webrepl服务，接下来我们会用到这个服务上传文件，实现自动连接WiFi热点。

```
>>> import network
>>> sta_if = network.WLAN(network.STA_IF)
>>> sta_if.active(True)
True
>>> sta_if.scan()
[...]
>>> sta_if.connect("你的WiFi", "你的密码")
>>> sta_if.isconnected()
True
>>> print('Network Configuration:', sta_if.ifconfig())
Network Configuration: ('192.168.3.148', '255.255.255.0', '192.168.3.1', '192.168.3.1')
>>> import webrepl
>>> webrepl.start()
WebREPL server started on http://192.168.3.148:8266/
Started webrepl in normal mode
```

注意webrepl必须设置密码才能访问，如果你之前没有设置密码，需要执行 `import webrepl_setup` 语句设置访问密码。设置好后让esp32重启一下，新密码就生效了。

```shell
>>> import webrepl_setup
WebREPL daemon auto-start status: enabled

Would you like to (E)nable or (D)isable it running on boot?
(Empty line to quit)
> E
Would you like to change WebREPL password? (y/n) y
New password (4-9 chars): xxxxxx
Confirm password: xxxxxx
No further action required
```


## 通过boot.py文件实现自动连接WiFi热点

MicroPython在启动时会先查看并执行boot.py文件，我们可以把连接WiFi的代码添加到这个文件里，就可以实现自动连接WiFi热点。我们要使用的boot.py内容如下，实现的功能就是连接WiFi和打开webrepl服务。

```python
# This file is executed on every boot (including wake-boot from deepsleep)
import webrepl
import esp

esp.osdebug(None)

def do_connect():
    import network

    SSID = '你的WiFi'
    PASSWORD = '你的密码'

    sta_if = network.WLAN(network.STA_IF)
    ap_if = network.WLAN(network.AP_IF)
    if ap_if.active():
        ap_if.active(False)
    if not sta_if.isconnected():
        print('connecting to network...')
        sta_if.active(True)
        sta_if.connect(SSID, PASSWORD)
        while not sta_if.isconnected():
            pass
    print('Network configuration:', sta_if.ifconfig())

do_connect()
webrepl.start()
```

我们下面打开浏览器输入 `http://<esp32_ip>:8266` ，将 `<esp32_ip>` 。界面如下，我们需要使用右边的"Send a file"功能上传我们的boot.py文件。 ![img](/images/micropython_webrepl_send_file.jpg) 先点击左边的"Connect"按钮连接ESP32，点击“浏览”按钮选择准备好的boot.py，再点击“Send to device”就可以将文件提交到ESP32上。这时如果再重启ESP32，ESP32就会自己连接WiFi热点了。一般来说WiFi热点会给同一个设备分配相同的IP，所以我们可以使用之前使用的IP来使用webrepl服务。


## 总结

今天我们介绍了怎么让ESP32自动连接WiFi热点，我们就可以将ESP32加入到我们的家庭网络里。下一篇文章我们会用ESP32和MQTT配合一起来控制电器开关。

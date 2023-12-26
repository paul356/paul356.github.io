---
layout: default
title: 创客利器之ESP32（四点五）- 使用ESP32和MQTT控制小灯 - 小插曲
tags: [esp32, MCU, micropython, MQTT]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 创客利器之ESP32（四点五）- 使用MicroPython与MQTT一起控制小灯 - 小插曲


## 小插曲

上一篇文章我们介绍了如何使用MicroPython连接MQTT Broker。因为我有两个ESP32，都使用了上一篇文章中的main.py。

    import time
    from umqtt.simple import MQTTClient
    from machine import Pin
    
        # Publish test messages e.g. with:
        # mosquitto_pub -t foo_topic -m hello
    
        server = "<mqtt_broker_ip>"
        port_num = 9001
        swith_topic = b"home/bedroom/light"
        p5 = Pin(5, Pin.OUT)
        p5.off()
    
        # 如果值是on，打开小灯；否则关掉小灯
        def sub_cb(topic, msg):
            if msg.decode().lower() == "on":
                p5.on()
            else:
                p5.off()
    
    def main():
        print("main started ...")
        c = MQTTClient("umqtt_client", server, port=port_num)
        c.set_callback(sub_cb)
        c.connect()
        c.subscribe(swith_topic)
        while True:
            try:
                if True:
                    # Blocking wait for message
                    c.wait_msg()
            except Exception:
                print("wait message fail")
        c.disconnect()
    
    print("will enter main")
    if __name__ == "__main__":
        main()

但是我发现只要我打开第二个ESP32时，第一个打开的ESP32就会报错，一直打印 `wait message fail` 。开始我怀疑是因为MQTT Broker不支持多个客户端订阅同一个地址，但是查了文档发现并没有这样的问题，而且使用mosquitto\_sub工具就没有这样的问题。在没有找到原因的情况下，就像很多程序员一样，先修改代码绕过问题。修改后的代码如下。结果两个ESP32每隔一秒钟就报一次错，然后重新连接，不完美还是可以打开关闭小灯。

    def main():
        print("main started ...")
        connect = True
        while True:
            try:
                if connect:
                    client_id = f"umqtt_client{random.randint(0, 1000)}"
                    c = MQTTClient("umqtt_client", server, port=port_num)
                    c.set_callback(sub_cb)
                    c.connect()
                    c.subscribe(switch_topic)
                    print("client ", client_id, " connects")
                    connect = False
                if True:
                    # Blocking wait for message
                    c.wait_msg()
            except Exception as e:
                 print("wait message fail, ", e)
                 c.disconnect()
                 time.sleep(1)
                 connect = True
        c.disconnect()

由于这个方案并不完美，我还是决定上网找一下是否有人遇到和我一样的问题。还真找到了，有个人遇到了和我一样[问题](https://stackoverflow.com/questions/36184490/mqtt-client-disconnects-when-another-client-connects-to-the-server)。原来Mosquitto为每个client\_id只保留一个连接，而我的main.py使用了相同的client\_id。所以第二个ESP32连接时，第一个ESP32会被断开。再修改main函数如下，给client\_id加上一个随机数，这样不同ESP32的client\_id大概率就不一样了。

    def main():
        print("main started ...")
        connect = True
        while True:
            try:
                if connect:
                    client_id = f"umqtt_client{random.randint(0, 1000)}"
                    c = MQTTClient(client_id, server, port=port_num)
                    c.set_callback(sub_cb)
                    c.connect()
                    c.subscribe(switch_topic)
                    print("client ", client_id, " connects")
                    connect = False
                if True:
                    # Blocking wait for message
                    c.wait_msg()
            except Exception as e:
                 print("wait message fail, ", e)
                 c.disconnect()
                 time.sleep(1)
                 connect = True
        c.disconnect()


## 总结

看来看清楚API文档很重要。


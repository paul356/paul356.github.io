---
layout: default
title: 使用Rust语言为ESP编程系列一 -- 准备开发环境（续）
tags: [Rust on ESP, Rust, ESP32]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 使用Rust语言为ESP编程系列一 &#x2013; 准备开发环境（续）


## 前言

前一篇文章[使用Rust语言为ESP编程系列一 &#x2013; 准备开发环境](https://paul356.github.io/2024/11/11/rust-on-esp-series_1.html)介绍了如何逐个组件安装Rust on ESP环境的方法，过程比较复杂。其实还有一种更加简便的方法，那就是使用乐鑫制作好的docker镜像。这一篇续就是介绍如何通过docker镜像来获得开发环境。


## 安装过程

使用docker需要先安装docker-ce，安装步骤大家可以自行搜索。另外docker-ce默认从docker.com下载镜像。如果出现下载速度的问题，可以修改成国内的docker仓库，具体步骤可以搜索网上教程。

1.  获取[idf-rust](https://hub.docker.com/r/espressif/idf-rust/tags)镜像。
    
    使用docker pull命令就可以下载idf-rust镜像。这个镜像有不同的tag分别表示针对不同的芯片和环境版本的镜像，我们使用了all\_lastest这个tag，可以支持全系列芯片。
    
    ```shell
    docker pull espressif/idf-rust:all_latest
    docker images
    ```
    
    最后使用 `docker images` 查看下载的镜像信息。

2.  创建idf-rust容器实例。
    
    使用 `docker run` 命令创建容器实例。将 `<code_path>` 替换成实际的代码目录。
    
    ```shell
    docker run -itd -v <code_path>:/code espressif/idf-rust:all_latest bash
    docker ps
    ```
    
    使用 `docker ps` 查看创建的实例ID和名称。下面的命令需要用到实例ID或名称。
    
    上一条命令创建了容器实例，但创建的容器处于后台运行的状态。需要使用容器时，需要通过下列命令。
    
    ```shell
    docker exec -it <实例ID或名称> bash
    ```
    
    环境安装好了，下面可以进去代码目录，执行 `cargo build` 命令编译Rust代码。需要注意的是在上一篇文章中，为了使用已安装好的ESP-IDF环境我们需要在Cargo.toml中增加了配置段 `[package.metadata.esp-idf-sys]` ，在idf-rust镜像中是不需要的。


## 总结

使用docker镜像使得Rust on ESP安装过程变得非常方便，所以花一点时间学习如何docker使用也是非常值得的。现在刀磨好了，我们下面就可以正式砍材了（真正开始探索Rust on ESP）。

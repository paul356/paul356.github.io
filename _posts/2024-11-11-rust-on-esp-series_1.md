---
layout: default
title: 使用Rust语言为ESP编程系列一 -- 准备开发环境
tags: [Rust on ESP, Rust, ESP32]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 使用Rust语言为ESP编程系列一 &#x2013; 准备开发环境


## 前言

之前尝试写了一篇文章[尝试使用Rust为esp系列控制器开发软件](file:///2024/10/31/mobile-storage.html)介绍了如何用Rust实现一个运行在ESP32S3上的webdav服务，发现还挺受欢迎，所有打算展开来讲一讲，希望能够对有兴趣在ESP上使用Rust语言的朋友有所帮助。

本文预设读者对使用Linux命令行、Rust开发和ESP系列控制器有一些基本认识，所以对如何切换目录和执行命令等操作细节，不做特别解释。


## 原理浅析

使用Rust语言为ESP芯片开发程序是在PC或服务器上利用交叉编译的方法，生成能够运行在ESP芯片上的代码。Rust on ESP交叉编译环境通过扩展通用Rust开发环境而实现。通过增加必要的扩展，使Rust代码可以被编译为可运行在ESP上的机器码；并提供针对ESP的代码库，使得可以在Rust代码中访问ESP系统的设备。甚至我们可以直接使用系统库std的功能，就像在PC上开发代码一样使用内存分配、创建用户线程、文件接口等标准库接口。我们也可以不选择使用系统库std，只使用Rust的核心库core，可以生成更小镜像，但也不能使用内存分配器、线程模型和文件系统等抽象接口，代码直接调用硬件驱动提供的接口。


## Linux环境下安装开发环境

如果你熟悉docker的使用，有一个更加简便的方法就是使用docker镜像[idf-rust](https://hub.docker.com/r/espressif/idf-rust/tags)，这个镜像已经包含一个安装好了的Rust开发环境。下面是安装Rust on ESP的详细过程。

1.  安装Rust on ESP开发环境 开发环境基于通用的Rust的开发环境，第一步和安装普通的Rust开发环境一样。如果是Linux环境，可以在命令行中运行[rustup](https://rustup.rs/)网站的安装命令。这个命令会自动下载Rust开发环境并安装。
    
    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```
    
    安装好Rust开发环境之后，下面就是安装ESP需要的Rust扩展部分，因为ESP系列分别有Xtensa核心和RISC-V核心的芯片。老款的ESP32、ESP32S2、ESP32S3是基于Xtensa核心的芯片，而新款的ESP32-C系列、ESP32-H系列和ESP32-P系列都是基于RISC-V核心。仅安装支持RISC-V核心芯片的工具链和安装支持两种核心芯片的工具链步骤有所不同。因为我主要使用ESP32，所以我选择第二个安装步骤。
    
    安装的命令其实很简单，主要的工作由espup工具完成。先安装espup，然后执行 `espup install` 。
    
    ```shell
    cargo install espup
    espup install
    ```
    
    espup会帮我们安装下列组件：
    
    -   支持ESP系列芯片的Rust
    
    -   支持RISC-V的工具链
    
    -   支持Xtensa核心的LLVM
    
    -   GCC工具链用于链接过程，并负责生成二进制文件
    
    安装过程可能需要花不少时间，一方面因为需要下载不少软件组件，另外还受到下载速度影响。如果网速不稳，经常遇到下载软件包失败的问题，可能需要通过代理或镜像来解决。网络问题是当前安装过程中的最大麻烦，其他过程还比较简单。

2.  安装系统库的依赖 成功完成上面的步骤，我们已经安装好了开发环境。但要使用Rust系统库std，我们还要安装std依赖的组件。面向ESP的系统库std有两个依赖，ESP-IDF开发框架和ldproxy库。
    
    先安装ESP-IDF开发框架，第一步安装ESP-IDF依赖的第三方工具。
    
    -   Ubuntu和Debian环境
        
        ```shell
        sudo apt-get install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
        ```
    
    -   CentOS 7 & 8
        
        ```shell
        sudo yum -y update && sudo yum install git wget flex bison gperf python3 cmake ninja-build ccache dfu-util libusbx
        ```
    
    -   Arch
        
        ```shell
        sudo pacman -S --needed gcc git make flex bison gperf python cmake ninja ccache dfu-util libusb
        ```
    
    下面安装ESP-IDF框架本身，先下载ESP-IDF的源代码。当前的稳定版本是5.3，所以这里选择代码分支release/v5.3，并更新所有子模块。
    
    ```shell
    mkdir -p ~/esp
    cd ~/esp
    git clone --recursive https://github.com/espressif/esp-idf.git
    git checkout release/v5.3
    git submodule update --init --recursive
    ```
    
    下载好代码后开始ESP-IDF安装过程，进入esp-idf目录，执行下面命令。第二条命令用于将下载源导向乐鑫科技为国内网络提供的资源，这样下载速度会更快。
    
    ```shell
    cd ~/esp/esp-idf/
    export IDF_GITHUB_ASSETS="dl.espressif.cn/github_assets"
    ./install.sh esp32,esp32s2,esp32s3
    ```
    
    安装好ESP-IDF框架后，再安装ldproxy库，这个比较简单。
    
    ```shell
    cargo install ldproxy
    ```
    
    到这里就完成了系统库std依赖的安装，系统库已经包含在espup安装的组件内了，所以系统库本身已经安装好了。


## 开发环境测试

我们创建Rust on ESP项目需要用到乐鑫提供的一个工具cargo-generate。先安装cargo-generate工具。

```shell
cargo install cargo-generate  
```

再使用 `cargo generate` 命令生成一个使用std的空项目。如不使用系统库，使用 `esp-generate --chip=<esp32xx> <your-project>` 命令。

```shell
cargo generate esp-rs/esp-idf-template cargo
```

这是一个向导式工具，填入项目名称、目标芯片、选择不配置高级选项。工具输出如下。

```text
user1@blackbox:~/code$ cargo generate esp-rs/esp-idf-template cargo
⚠️   Favorite `esp-rs/esp-idf-template` not found in config, using it as a git repository: https://github.com/esp-rs/esp-idf-template.git
🤷   Project Name: hello-world-rust
🔧   Destination: /home/user1/code/hello-world-rust ...
🔧   project-name: hello-world-rust ...
🔧   Generating template ...
✔ 🤷   Which MCU to target? · esp32
✔ 🤷   Configure advanced template options? · false
🔧   Moving generated files into: `/home/user1/code/hello-world-rust`...
🔧   Initializing a fresh Git repository
✨   Done! New project created /home/user1/code/hello-world-rust
```

打开Cargo.toml，在最后面增加下面的一段。告诉编译模块不要再下载ESP-IDF，使用环境变量中指定的ESP-IDF。

```text
[package.metadata.esp-idf-sys]
esp_idf_tools_install_dir = "fromenv"
esp_idf_sdkconfig = "sdkconfig"
esp_idf_sdkconfig_defaults = ["sdkconfig.defaults", "sdkconfig.defaults.ble"]
```

在src/main.rs中添加代码，这里测试了动态内存分配。

```rust
fn main() {
    // It is necessary to call this function once. Otherwise some patches to the runtime
    // implemented by esp-idf-sys might not link properly. See https://github.com/esp-rs/esp-idf-template/issues/71
    esp_idf_svc::sys::link_patches();

    // Bind the log crate to the ESP Logging facilities
    esp_idf_svc::log::EspLogger::initialize_default();

    let five = Box::new(5);

    log::info!("Hello, world! Give me {}", *five);
}
```

下面就可以编译代码了，在编译前一定要执行下面命令，用于定义Rust on ESP和ESP-IDF的一些环境变量。

```shell
. $HOME/export-esp.sh
. $HOME/esp/esp-idf/export.sh
```

第一个是Rust on ESP开发环境的环境变量，第二个是ESP-IDF的一些环境变量。 **顺序** 一定不能反，我定位了好久才发现这个问题。由于espup安装的交叉编译工具和ESP-IDF自带的交叉编译工具版本有所不同，所以要让ESP-IDF环境变量覆盖前一个工具的环境变量，不然编译过程会报告版本不对。然后运行 `cargo build` 命令。

```shell
cargo build
```

如果安装过程没有问题，会生成固件文件hello-world-rust, 位置在 `target/xtensa-esp32-espidf/debug/hello-world-rust` 。用usb线连接ESP32开发板，运行 `cargo espflash flash` 将固件刷到板子上。如果没有espflash，先安装 `cargo install espflash` 。再运行 `cargo espflash monitor` 查看输出，如果没有问题，这时你会看到下面的输出。

```text
...
I (423) main_task: Started on CPU0
I (433) main_task: Calling app_main()
I (433) hello_world_rust: Hello, world! Give me 5
I (433) main_task: Returned from app_main()
```


## 总结

恭喜你完成了Rust on ESP的安装，现在你就可以用Rust给ESP系列开发板写代码了。下面我们会进一步探索使用Rust语言我们可以做出哪些有趣的尝试。本文主要参考的互联网上的资料，其中关于Rust on ESP开发环境的安装主要参考的[The Rust on ESP Book](https://docs.esp-rs.org/book/)，ESP-IDF的安装参考的乐鑫提供的[安装指导](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/get-started/linux-macos-setup.html)。安装过程繁琐，难免有所疏漏，如果大家发现问题欢迎大家指正。

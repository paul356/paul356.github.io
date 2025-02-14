---
layout: default
title: 一种基于ESP32丰富连接能力的移动存储设备 -- 测试TF读写速度
tags: [Rust on ESP, ESP32, SPI, SDMMC， TFCARD]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 一种基于ESP32丰富连接能力的移动存储设备 &#x2013; 测试TF读写速度


## 前言

在前面的文章中我已经验证了TF卡读写功能，证实了我们可以通过ESP-IDF的接口访问FAT格式的TF卡。为了验证通过WebDAV协议访问TF卡的体验，我将WebDAV前端和TF卡访问功能集成到了一起，使得WebDAV模块可以从TF卡挂载的位置读取文件。本文是我的测试过程。

本系列其他文章

1.  [基于ESP32的移动存储设备](https://paul356.github.io/2024/10/31/mobile-storage.html)
2.  [一种基于ESP32丰富连接能力的移动存储设备 &#x2013; 电路设计](https://paul356.github.io/2024/12/12/mobile-storage-pcb.html)
3.  [一种基于ESP32丰富连接能力的移动存储设备 &#x2013; 测试SD卡读写](https://paul356.github.io/2024/12/27/mobile-storage-sd-card-test.html)
4.  [一种基于ESP32丰富连接能力的移动存储设备 &#x2013; 验证屏幕显示](https://paul356.github.io/2025/01/06/mobile-storage-display.html)


## 测试micro-storage

本文使用的代码都可以在代码仓库[esp-webdav](https://github.com/paul356/esp-webdav)中找到。在电脑上可以访问运行起来的WebDAV服务。

![img](/images/esp_webdav_screenshot.png)


### 测试顺序读取速度

我们希望测得TF卡的顺序读取速度，因为通过WebDAV协议访问基本上不会有随机读取的场景。为了测试无干扰下的TF卡顺序读取速度，我写了一个顺序读取一个大文件的函数。

```Rust
async fn test_file_perf(mount_point: &str) -> anyhow::Result<()> {
    let file_path = format!("{}/habanera.mp4", mount_point);

    tokio::task::spawn(async move {
        let mut file = tokio::fs::File::open(file_path).await?;
        let mut read_buf = std::vec![0;32*1024];
        let mut read_size: usize = 0;

        let beg = tokio::time::Instant::now();

        for _i in 0..400 {
            let size = file.read(&mut read_buf).await?;
            read_size += size;
        }

        info!("[Async] Read {} bytes in {:?}", read_size, beg.elapsed());

        Ok::<_, anyhow::Error>(())
    }).await?
}

fn test_file_sync(mount_point: &str) -> anyhow::Result<()> {
    let file_path = format!("{}/habanera.mp4", mount_point);
    let mut file = std::fs::File::open(file_path)?;
    let mut read_buf = std::vec![0;32*1024];
    let mut read_size: usize = 0;

    let beg = tokio::time::Instant::now();

    use std::io::Read;
    for _i in 0..400 {
        let size = file.read(&mut read_buf).unwrap();
        read_size += size;
    }

    info!("[Sync] Read {} bytes in {:?}", read_size, beg.elapsed());

    Ok::<_, anyhow::Error>(())
}

```

用于测试的文件位于TF根目录下，大约有100MB。测试方法很简单，每次读取32KB的文件内容，连续请求400次，最后获取消耗的时间。

```text
I (1709) micro_storage: [Sync] Read 13107200 bytes in 1.593215s
...
I (21727) micro_storage: [Async] Read 13107200 bytes in 4.229245s
```

我共测试了两次，第一次使用同步接口，第二次使用了tokio库的异步接口。使用异步接口后同样的任务时间却增加了2.6秒，大概由于只有一个并发请求，增加了开销但没有带来收益。但增加的时延具体耗费在哪些事情上了，还需要进一步分析。对于ESP32这样的资源受限的平台，使用异步编程接口可能是”杀鸡用了牛刀“。同步读的速度约为7.9MB/秒，异步读的速度约为3.0MB/秒。


### 测试顺序写速度

测试顺序写速度使用了和测试顺序读类似的方法。而且也测试了一次同步接口和一次异步接口。

```Rust
async fn test_wfile_perf(mount_point: &str) -> anyhow::Result<()> {
    let file_path = format!("{}/wfile_async.bin", mount_point);

    tokio::task::spawn(async move {
        let mut file = tokio::fs::File::create(file_path).await?;
        let mut write_buf = std::vec![0xfu8;32*1024];
        let mut write_size: usize = 0;

        let beg = tokio::time::Instant::now();

        for _i in 0..400 {
            let size = file.write(&mut write_buf).await?;
            write_size += size;
        }

        info!("[Async] Write {} bytes in {:?}", write_size, beg.elapsed());

        Ok::<_, anyhow::Error>(())
    }).await?
}

fn test_wfile_sync(mount_point: &str) -> anyhow::Result<()> {
    let file_path = format!("{}/wfile_sync.bin", mount_point);
    let mut file = std::fs::File::create(file_path)?;
    let mut write_buf = std::vec![3u8;32*1024];
    let mut write_size: usize = 0;

    let beg = tokio::time::Instant::now();

    use std::io::Write;
    for _i in 0..400 {
        let size = file.write(&mut write_buf).unwrap();
        write_size += size;
    }

    info!("[Sync] Write {} bytes in {:?}", write_size, beg.elapsed());

    Ok::<_, anyhow::Error>(())
}
```

测试结果如下。经过计算同步写的速度大约为3MB/秒，而异步写速度只有1.8MB/秒。

```text
I (6263) micro_storage: [Sync] Write 13107200 bytes in 4.11604s
...
I (29223) micro_storage: [Async] Write 13107200 bytes in 7.079895s
```


## 总结

我们得到同步顺序读写的速度为7.9MB/秒和3MB/秒，异步接口的顺序读写性能要差一些，为3MB/秒和1.8MB/秒。前一对数字还差强人意，后一对数字就有点难看了。但是我们的WebDAV程序正是使用的异步接口，所以读写性能不会太好。本来想实际测试WebDAV程序的下载和上传性能，但是因为系统经常报告内存不够，导致系统OOM。虽然我的ESP32S3模块有8MB的SPIRAM，但是我发现启用了SPIRAM之后，会出现无法挂载TF卡的错误，我怀疑是硬件设计的问题，还需要等优化了SDMMC硬件后再次测试。


## 链接

1.  esp\_webdav - <https://github.com/paul356/esp-webdav>

---
layout: default
title: 一种基于ESP32丰富连接能力的移动存储设备 -- 测试SD卡读写
tags: [Rust on ESP, ESP32, PCB, SD Card]
nav_order: {{ page.date }}
sync_wexin: 1
---


# 一种基于ESP32丰富连接能力的移动存储设备 &#x2013; 测试SD卡读写


## 前言

上一篇文章介绍了我为移动存储设备设计的一块电路板，并且PCB实物也收到了。现在我已经焊接好了电路板，接下来我想先测试SD卡的读写能力是否正常。 ![img](/images/esp32-storage-assembled-pcb.jpg)

本系列其他文章

1.  [基于ESP32的移动存储设备](https://paul356.github.io/2024/10/31/mobile-storage.html)
2.  [一种基于ESP32丰富连接能力的移动存储设备 &#x2013; 电路设计](https://paul356.github.io/2024/12/12/mobile-storage-pcb.html)


## 测试SD卡功能


### 使用官方例子

要快速测试SD卡的功能，我先找了ESP-IDF 5.3中的一个C语言例子，例子在ESP-IDF源码中的这个位置 `examples/storage/sd_card/sdmmc` 。例子中的SD卡槽连接的管脚和我的PCB不一样，需要通过 `idf.py menuconfig` 修改CLK，CMD等管脚对应的GPIO口编号。修改之后的状态如下。 ![img](/images/sdmmc-menuconfig.png)

然后使用 `idf.py build` 命令编译代码，使用 `idf.py flash` 上传固件。插入SD卡，测试很顺利。

```text
I (897) example: Reading file /sdcard/foo.txt
I (897) example: Read from file: 'Hello SL16G!'
I (897) example: Opening file /sdcard/nihao.txt
I (1117) example: File written
I (1117) example: Reading file /sdcard/nihao.txt
I (1117) example: Read from file: 'Nihao SL16G!'
I (1117) example: Card unmounted
I (1127) main_task: Returned from app_main()
```


### 开发Rust代码

通过官方例子验证了我的SD卡硬件设计没有问题，下面就让我们通过Rust来访问SD卡。虽然通过搜索网路我找到一个Rust模块embeded-sdmmc，但是这个项目目前只支持通过SPI协议连接的SD卡。因为我为了获取更好的读写速度，选用了SDMMC接口，所以我无法使用这个模块。之后搜索也没有找到其他支持SDMMC的模块。所以只能直接调用ESP-IDF的C接口这个办法了。

通过查看官方例子的代码，我们了解到关键的接口是 `esp_vfs_fat_sdmmc_mount` 函数。

```c
/**
 * @brief Convenience function to get FAT filesystem on SD card registered in VFS
 *
 * This is an all-in-one function which does the following:
 * - initializes SDMMC driver or SPI driver with configuration in host_config
 * - initializes SD card with configuration in slot_config
 * - mounts FAT partition on SD card using FATFS library, with configuration in mount_config
 * - registers FATFS library with VFS, with prefix given by base_prefix variable
 *
 * This function is intended to make example code more compact.
 * For real world applications, developers should implement the logic of
 * probing SD card, locating and mounting partition, and registering FATFS in VFS,
 * with proper error checking and handling of exceptional conditions.
 *
 * @note Use this API to mount a card through SDSPI is deprecated. Please call
 *       `esp_vfs_fat_sdspi_mount()` instead for that case.
 *
 * @param base_path     path where partition should be registered (e.g. "/sdcard")
 * @param host_config   Pointer to structure describing SDMMC host. When using
 *                      SDMMC peripheral, this structure can be initialized using
 *                      SDMMC_HOST_DEFAULT() macro. When using SPI peripheral,
 *                      this structure can be initialized using SDSPI_HOST_DEFAULT()
 *                      macro.
 * @param slot_config   Pointer to structure with slot configuration.
 *                      For SDMMC peripheral, pass a pointer to sdmmc_slot_config_t
 *                      structure initialized using SDMMC_SLOT_CONFIG_DEFAULT.
 * @param mount_config  pointer to structure with extra parameters for mounting FATFS
 * @param[out] out_card  if not NULL, pointer to the card information structure will be returned via this argument
 * @return
 *      - ESP_OK on success
 *      - ESP_ERR_INVALID_STATE if esp_vfs_fat_sdmmc_mount was already called
 *      - ESP_ERR_NO_MEM if memory can not be allocated
 *      - ESP_FAIL if partition can not be mounted
 *      - other error codes from SDMMC or SPI drivers, SDMMC protocol, or FATFS drivers
 */
esp_err_t esp_vfs_fat_sdmmc_mount(const char* base_path,
                                  const sdmmc_host_t* host_config,
                                  const void* slot_config,
                                  const esp_vfs_fat_mount_config_t* mount_config,
                                  sdmmc_card_t** out_card);
```

这个函数接收一个文件系统路径、SD驱动配置、管脚配置、挂载配置，输出SD卡对象的句柄。我们只要准备这些配置对象，就可以读写SD卡了。由于在官方的C例程中，使用了一些宏来初始化 `sdmmc_host_t` 和 `sdmmc_slot_config_t` ，这些宏在Rust中访问不到，所以我们需要参照C的源代码来初始化这些结构体。

通过Rust初始化 `sdmmc_slot_config_t` 结构体，管脚的赋值需要依据电路设计。

```Rust
  fn get_slot_config() -> sdmmc_slot_config_t {
    sdmmc_slot_config_t {
        clk: 7,
        cmd: 6,
        d0: 15,
        d1: 16,
        d2: 4,
        d3: 5,
        d4: -1,
        d5: -1,
        d6: -1,
        d7: -1,
        __bindgen_anon_1: sdmmc_slot_config_t__bindgen_ty_1 {
            cd: 17,
        },
        __bindgen_anon_2: sdmmc_slot_config_t__bindgen_ty_2 {
            wp: -1,
        },
        width: 4,
        flags: SDMMC_SLOT_FLAG_INTERNAL_PULLUP,
    }
}
```

构造参数mount\_config、mount\_point、host\_config。

```Rust
let sdmmc_mount_config = esp_vfs_fat_sdmmc_mount_config_t {
    format_if_mount_failed: false,
    max_files: 4,
    allocation_unit_size: 16 * 1024,
    disk_status_check_enable: false,
    use_one_fat: false,
};

let mount_point = CString::new("/sdcard").unwrap();

let sd_host = sdmmc_host_t {
    flags: SDMMC_HOST_FLAG_1BIT | SDMMC_HOST_FLAG_4BIT | SDMMC_HOST_FLAG_8BIT | SDMMC_HOST_FLAG_DDR,
    slot: SDMMC_HOST_SLOT_1,
    max_freq_khz: SDMMC_FREQ_DEFAULT,
    io_voltage: 3.3,
    init: Some(sdmmc_host_init),
    set_bus_width: Some(sdmmc_host_set_bus_width),
    get_bus_width: Some(sdmmc_host_get_slot_width),
    set_bus_ddr_mode: Some(sdmmc_host_set_bus_ddr_mode),
    set_card_clk: Some(sdmmc_host_set_card_clk),
    set_cclk_always_on: Some(sdmmc_host_set_cclk_always_on),
    do_transaction: Some(sdmmc_host_do_transaction),
    __bindgen_anon_1: sdmmc_host_t__bindgen_ty_1 {
        deinit: Some(sdmmc_host_deinit)
    },
    io_int_enable: Some(sdmmc_host_io_int_enable),
    io_int_wait: Some(sdmmc_host_io_int_wait),
    command_timeout_ms: 0,
    get_real_freq: Some(sdmmc_host_get_real_freq),
    input_delay_phase: SDMMC_DELAY_PHASE_0,
    set_input_delay: Some(sdmmc_host_set_input_delay),
    dma_aligned_buffer: std::ptr::null_mut(),
    pwr_ctrl_handle: std::ptr::null_mut(),
    get_dma_info: Some(sdmmc_host_get_dma_info),
};

let slot_config = get_slot_config();
```

参数都准备好了，最后调用esp\_vfs\_fat\_sdmmc\_mount挂载文件系统。

```Rust
let mut card_handle: *mut sdmmc_card_t = std::ptr::null_mut();

let ret = unsafe {
    esp_vfs_fat_sdmmc_mount(
        mount_point.as_ptr(),
        &sd_host,
        &slot_config as *const sdmmc_slot_config_t as *const c_void,
        &sdmmc_mount_config,
        &mut card_handle,
    )
};

match ret {
    ESP_OK => log::info!("SD Card mounted"),
    _ => {
        log::error!("Failed to mount SD Card");
        return;
    }
}
```

再测试SD卡的基本文件读写功能。

```Rust
let file_res = std::fs::File::create_new("/sdcard/test.txt");
let mut file = match file_res {
    Ok(mut file) => {
        file.write_all(b"Hello, world!").unwrap();

        std::fs::File::open("/sdcard/test.txt").unwrap()
    }
    Err(_) => {
        std::fs::File::open("/sdcard/test.txt").unwrap()
    }
};

let mut content = String::new();
file.read_to_string(&mut content).unwrap();

log::info!("File content: {}", content);
```

日志如下，初步看来SD卡功能正常。

```text
I (417) main_task: Started on CPU0
I (427) main_task: Calling app_main()
I (427) sd_card_test: Hello, world!
I (427) gpio: GPIO[7]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
I (437) gpio: GPIO[6]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
I (447) gpio: GPIO[15]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
I (457) gpio: GPIO[16]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
I (467) gpio: GPIO[4]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
I (477) gpio: GPIO[5]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0 
I (517) gpio: GPIO[5]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
I (527) sd_card_test: SD Card mounted
I (527) sd_card_test: File content: Hello, world!
I (527) main_task: Returned from app_main()
```

以上就是Rust中使用SD卡的关键要点，其实还是比较简单的。


## 后记

有兴趣的朋友可以在[sd\_card\_rust](https://github.com/paul356/sd_card_rust)项目中找到完整代码，欢迎大家留言交流。


## 链接列表

1.  sd\_card\_rust &#x2013; <https://github.com/paul356/sd_card_rust>

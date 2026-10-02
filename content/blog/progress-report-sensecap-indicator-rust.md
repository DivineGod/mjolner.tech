+++
title = "Progress Report: SenseCAP Indicator + Rust"
slug = "progress-report-sensecap-indicator-rust"
date = "2026-02-21T09:23:00+00:00"
include_in_feeds = false
discoverable = false
is_page = false
+++

The past few weeks I've been tinkering with the [Seeed Studio SenseCAP Indicator][sensecap-wiki], a very sensor platform with LoRa radio, powered by an ESP32-S3 and an RP2040. The device also has a 4-inch touch screen controlled by the ESP32 chip.

# Goal

My ultimate goal is to be able to develop a rust based monitoring station that can show sensor data as well as using LoRa to communicate with LoRa nodes.

# Why rust?

I like rust and I like using it to understand things.

Also, I tried running the examples from Seeed Studio on the device and the ones I wanted to use to test the display didn't even compile with the proscribed esp-idf versions. I did try to get it to work, but not to the point of really getting into it.

# Progress

 - [x] Test Rust on esp32 mcu
 - [ ] Test Rust on rp2040 mcu
 - [x] Make the display work
 - [ ] Write an embedded-graphics interface for the st7701s
 - [ ] Find more things to put on the to-do list

# Getting Rust working on the ESP32

Getting rust to run on the ESP32 is straightforward these days following the Espressif documentation and using `esp-generate` to create a sample project gets you up and running very quickly.

# Interfacing with st7701s on the SenseCAP Indicator

The SenseCAP Indicator's interface with the st7701s display driver chip is through "3-wire" SPI for commands and Parallel RGB for display data.

The ESP32 has a specific peripheral device built-in for dealing with the display data, it's the `LCD_CAM` peripheral.

For this particular use-case the [`Dpi`][dpi-docs-rs] (Digital Parallel Interface) device must be used. 

## Setting up Dpi

There aren't any examples or tutorials on how to use the Dpi device, but I did find a [qa-test][lcd-dpi-qa-esp-hal] for it

Next I had to figure out the correct configuration values. Since the only reference I really could find were using esp-idf, I trawled through the source for esp-idf to find out how the rust structure mapped to the low level commands utilised by the esp-idf library.
I came upon the timing values for the display, they didn't directly map to the expected values in the Rust library, so I had to reverse engineer the values based off the timings from the c implementation and the way the low-level esp-idf code mapped the values and the way the esp-hal Dpi configuration code was using the values, as they ultimately use the same peripherals.

### Frame Timing

[Frame-timing calculations in esp-idf][frame-timing-calc]

[Frame timing definition][sensecap-timings] in the SenseCAP Indicator Template

```rust
    let hsw = 8; // Horizontal Sync Width
    let hbp = 50; // Horizontal Back Porch
    let hfp = 10; // Horizontal Front Porch
    let vsw = 8; // Vertical Sync Width
    let vbp = 20; // Vertical Back Porch
    let vfp = 10; // Vertical Front Porch
    let active_height = 480;
    let active_width = 480;

    let config = Config::default()
        .with_timing(FrameTiming {
            horizontal_active_width: active_width,
            horizontal_total_width: hsw + hbp + active_width + hfp,
            horizontal_blank_front_porch: hbp + hsw,

            vertical_active_height: active_height,
            vertical_total_height: vsw + vbp + active_height + vfp,
            vertical_blank_front_porch: vbp + vsw,

            hsync_width: hsw,
            vsync_width: vsw,

            hsync_position: 0,
        });
```

### Clock Frequency

Finding the clock frequency to set in the Dpi config had me searching through to the [KConfig.projbuild][kconfig-freq] file, where the default value for the pclk frequency is defined.

```rust
    let config = Config::default()
        .with_frequency(Rate::from_mhz(18));
```

### Colour Format

For the format I just needed to set it to 2-byte mode.

```rust
    let config = Config::default()
        .with_format(Format {
            enable_2byte_mode: true,
            ..Default::default()
        });
```

### Idle Values

This one was really annoying, and I'm not sure why this worked.

Reading through the rust code, the defaults are one thing and in the c sdk esp-idf they're a different thing.

I'm still not sure why the values I've gotten to work. I just tried out all the 8 different combinations of vsync, hsync, and de idle levels.

```rust
    let config = Config::default()
        .with_vsync_idle_level(Level::High)
        .with_hsync_idle_level(Level::High)
        .with_de_idle_level(Level::Low);
```



## Initialisation routine with 3-wire SPI

This was the bulk of the work to try to understand what the Seeed Studio SenseCAP Indicator template was doing to initialise the board. It also turned out the transformation from C to Rust was the most fraught one I've done yet.

### Run Clippy!

I found one of the mistakes I made, which was in the code for a while, was related to a loop to send the initialisation commands to the display.

`for _ in [0..9]` may look correct and rustc will not complain, the issue is that it generates a single iteration with the values of the range. I realised that it should be `for _ in 0..9` which correctly iterates 9 times.

Had I employed `cargo clippy` during the development and debugging, this could have saved me a few days worth of agonising over why things wouldn't work.



[sensecap-wiki]: https://wiki.seeedstudio.com/Sensor/SenseCAP/SenseCAP_Indicator/Get_started_with_SenseCAP_Indicator/
[dpi-docs-rs]: https://docs.espressif.com/projects/rust/esp-hal/1.0.0/esp32s3/esp_hal/lcd_cam/lcd/dpi/index.html
[lcd-dpi-qa-esp-hal]: https://github.com/esp-rs/esp-hal/blob/main/qa-test/src/bin/lcd_dpi.rs
[frame-timing-calc]: https://github.com/espressif/esp-idf/blob/ffb63db38b9d59929b6d2600af0f6a11deea13ac/components/esp_hal_lcd/esp32s3/include/hal/lcd_ll.h#L639
[sensecap-timings]: https://github.com/Seeed-Solution/indicator-esp-idf-template/blob/54f87462a7bacd2bed0afb9fc677626c5361b400/components/bsp/src/boards/sensecap_indicator_board.c#L132-L139
[kconfig-freq]: https://github.com/Seeed-Solution/SenseCAP_Indicator_ESP32/blob/77edb8d2b9a92fc67965c1b2d4a838f0d09a1800/components/bsp/Kconfig.projbuild#L102

+++
title = "Quick Note: ESP32 multicore"
slug = "quick-note-esp32-multicore"
date = "2025-01-23T23:00:00+00:00"
tags = ["embedded", "esp32", "multicore", "rust"]
include_in_feeds = true
discoverable = true
is_page = false
+++

Hi all!

Just wanted to let you all know that I'm busy working on a detailed post on developing for multicore embedded MCUs.

I am not entirely certain of when the post will be done, but the first working example is ready for public viewing and I thought I may as well show it off, so that while I'm working on getting more examples ready and finishing writing the post, people can start benefitting from these working examples.

The first example is using the ESP32C6 devkit from espressif. This module has two RISC-V cores. One running at up-to 140MHz and the other at 20MHz.

At this time, the example uses a shared memory location to communicate from the low-power core to the high-power core, which in turn logs the result it reads to a connected debug probe using defmt. It also tries to blink an LED connected to GPIO 1.

[ESP32C6 blinky dual-core example on GitHub][esp32c6-blinky]. Note that the crates have to be compiled separately. First compile `blinky-lp` then `blinky-hp`. The crates cannot currently be in a cargo workspace.

[esp32c6-blinky]: tab:https://github.com/DivineGod/multicore-mcu-examples/tree/main/esp32c6/blinky

+++
title = "Ratatui Pi"
slug = "ratatui-pi"
date = 2025-09-01T00:22:00+00:00
include_in_feeds = true
discoverable = true
is_page = false
+++

I went to the [Rust Forge Conference][rust_forge_conf] in Aotearoa New Zealand as a speaker, and because I was extremely excited that there finally was a Rust focused conference in this part of the world.

There, I presented my talk about writing [embedded devices drivers in Rust][rfc_talk_ando], which of course was based on my [blog post on the same topic][embedded_driver_blog].

At the conference [Orhun Parmaksız][orhun_gh] also did a workshop and a talk about his work on and with [Ratatui.rs][ratatui_rs]. The talk specifically was about Orhun's work with the ESP32 embedded version of ratatui in making a guitar-pedal-like system called [Tuitar][tuitar].

This inspired me to try integrating Ratatui with a touch display to control the user interface. Now while I did have an ESP32 based version of the display with me at the conference, it was going to be my demo device, so I wasn't keen on using it for testing the firmware in case I ended up breaking something too badly. As a backup I had also brought the RP2040 based version of the display with me, so I started looking into what was needed to get this running.

## `std` or `no_std` – why not both?

Espressif, the company behind the ESP32 ecosystem of microcontrollers, has been very good at embracing Rust over the past couple of years and they have managed to create to separate frameworks for integrating rust with their chips.

They have created a really great book called [The Rust on ESP book][esp_rust_book] and in the overview lists two ways of interacting with their systems. Either using the standard library or without. The standard way for most embedded systems is to avoid the standard library, because it requires support for features that usually are too much overhead for small systems like this. On the other hand, supporting a std environment does go a long way to help people transition into using embedded systems for developing solutions.

## RP2040 and ratatui

The ratatui community has already developed a solution for running on embedded systems. The crate [mousefood][mousefood] by [j-g00da][j-g00da_gh] was needed to provide the backend implementation for Ratatui to run on embedded devices. This crate had been ported to `no_std` recently so everything looked good to go.

I started out by copying the example I already had running from earlier and started adding `ratatui` and `mousefood` to `Cargo.toml` and adding example code to get a minimal example running.

Immediately I was hit with some issues. Besides being `no_std`, the RP2040 chip lacks a few features, such as atomic instructions, and `cargo build` was not happy. The [`kasuari`][kasuari] dependency (a crate implementing a 2D layout constraint solver) was throwing an error about `alloc::sync` missing.

## Rust Forge Community Spirit

As I was doing these investigations on eve of my talk, I didn't have time to do further investigations and so I posted in the Rust Forge discord channel, that I had started looking into this and if anyone wanted to get nerdshiped into fixing it, I would be more than happy to lend them my board to test out. To my amazement someone did respond! What was more impressive was that they had already brought their own RP2040 devices along to the conference.

[9names][9names_gh] quickly managed to post a fix for kasuari to their own fork of the crate. Fixing the dependency made the examples compile!

## Heaps progress

We were in business – almost. While I was busy preparing myself for my talk, 9names was busy looking at getting the examples running on the system.

Though the project was compiling correctly, the firmware seemed to crash on start-up and it turns out there needed to be a section setting up a specific amount of memory on the heap for it all to work. Soon enough 9names posted the following picture in the chat:

![Ratatui with mousefood running on a Raspberry Pi Pico 2 with a rectangular display. The screen says, in very small writing, Mousefood
Ratatui on embedded
devices](https://bear-images.sfo2.cdn.digitaloceanspaces.com/voided/pico2_mousefood.webp)

A huge win! It was running on the successor to the RP2040, the RP2350 chip. A more capable chip in many regards but it was still good to see progress.

Less than an hour later another picture was posted by 9names:

![The same code now running on the Round LCD Touch board with rp2040](https://bear-images.sfo2.cdn.digitaloceanspaces.com/voided/rat_round.webp)

The code was running on the RP2040 based round LCD Touch display! The only slight wrinkle in the story was that this time the heap allocation up front had to be increased.

## Integrating Touch

We now had a working prototype to run Ratatui with Mousefood on the RP2040 board and I set out to integrate my touch driver implementation into a simple example that demonstrated how to use both together.

I decided that implementing a simple counter would make sense. I could use the features of Ratatui to keep persistent data in an `App` struct and implement the `Widget` trait to render the counter.

The example also shows how to separate the App logic from the main board instantiation logic by having the core app logic in `src/app.rs`.

I put the finishing touches on the example as I was sitting in the Wellington Airport Lounge waiting for my flight back home. A great way to end a great conference.

This example is now part of my [device-driver-example repository][device_driver_gh] on GitHub.

[rust_forge_conf]: https://rustforgeconf.com/
[rfc_talk_ando]: https://slides.mjolner.tech/embedded-driver
[embedded_driver_blog]: https://blog.mjolner.tech/device-driver-rust/
[orhun_gh]: https://github.com/orhun/
[ratatui_rs]: https://ratatui.rs/
[tuitar]: https://github.com/orhun/tuitar/
[rust_esp_book]: https://docs.espressif.com/projects/rust/book/introduction.html
[mousefood]: https://github.com/j-g00da/mousefood/
[j-g00da_gh]: https://github.com/j-g00da/
[9names_gh]: https://github.com/9names/
[lcd_round_rat]: https://github.com/9names/rp2040_rust_playground/tree/rp2040_ratatui/lcd_round_rat/
[kasuari]: https://github.com/ratatui/kasuari
[kasuari_portable_atomics_pr]: https://github.com/ratatui/kasuari/pull/30
[device_driver_gh]: https://github.com/DivineGod/device-driver-example/tree/main/examples/lcd_round_rat

+++
title = "Exploring Multi-Core Microcontroller"
slug = "exploring-multi-core"
date = "2025-01-11T11:26:00+00:00"
include_in_feeds = false
discoverable = true
is_page = false
+++

# Introduction

 > It is becoming more common to have two or more cores in embedded processors, which adds an extra layer of complexity to concurrency. All the examples using a critical section [ ... ] assume the only other execution thread is the interrupt thread, but on a multi-core system that's no longer true. Instead, we'll need synchronisation primitives designed for multiple cores (also called SMP, for symmetric multi-processing).
 >
 > These typically use the atomic instructions we saw earlier, since the processing system will ensure that atomicity is maintained over all cores.
 >
 > Covering these topics in detail is currently beyond the scope of this book, but the general patterns are the same as for the single-core case.
 > [Rust Embedded Book][rust-embedded-book]

This certainly has a bit of "draw the rest of the owl"-energy. So let's look through our pencil case of tools to help us out. As outline in the chapter containing the quote above.

In embedded land when we’re just starting from scratch we might end up with a microcontroller that has more than one core. This really wasn’t much of the hobbyists' problem until recently. The list of microcontrollers with more than one core is growing quick.

With one core we already have a bunch to keep track of to not step on our own toes keeping access to memory and peripherals safe especially as the code grows more complex.

# Terms you may come across in the world of multicore microcontrollers

## Core Architecture Archetypes

### Homogeneous Cores
These chips have cores that are the same in terms of instruction sets and access to peripherals. In theory a task could be loaded from memory and executed on any of the cores without any trouble.

### Heterogeneous Cores
Heterogeneous cores are when all the cpu cores that are user programmable are of the same type, have access to the same functionality across the board.

## Processing Symmetry

Symmetricity describes the way the cores in a microcontroller is being utilised to run tasks. There are two main splits. Symmetric (SMP) and Asymmetric (AMP) multiprocessing.

### Symmetrical Multiprocessing

With SMP each cpu core is treated equally and have the same access to system resources.

- Usually requires homogeneous cores.
- CPUs share memory space (or, at least, some of it)
- normally an OS is used and this is a single instance that runs on all the CPUs, dividing work between them
- some kind of communication facility between the CPUs is provided (and this is normally shared memory)

SMP is generally considered good for compute intensive workloads where sharing processing power across cores can help parallelise a task.

### Asymmetric  Multiprocessing

In the asymmetric multiprocessing scheme each core might be of a different type and have a completely separate set of system resources available. Each task is isolated to running on a single core and don't usually share compute with other cores.

- Can be used with homogeneous or heterogenous cpus.
- each has its own memory address space
- some kind of communication facility between the CPUs is provided

AMP is typically good for systems with heterogeneous cores as each core can have a specific role to play depending on the system resources available. For instance the App/Net core setup on the nrf5340 chip has the net-core solely responsible for communicating with the radio sub-system and then messaging the App-core to do work.

## Embedded Operating Systems

The terms and concepts of SMP and AMP is mostly used when talking about operating systems. When we write applications for embedded systems we can either do so "bare metal" or use some type of operating system.

There exists a few embedded operating systems for rust. An OS in the embedded sense usually works a little different to your everyday OS. As everything is more restricted on embedded, applications are usually never dynamically loaded. Instead, the firmware is built statically, and the OS is a framework for building applications with less hassle. Depending on the goal of the framework, these affordances are multitude.

Rust based Real-time Operating Systems:

 - [DroneOS][droneos]
 - [Tock][tock]
 - [Hubris][hubris]/[Exhubris][exhubris]

 There also exists interfaces for Rust for [FreeRTOS][freertos-rust] and it's possible to write applications in Rust for [RIOT-OS][riot-os]

 Rust based embedded . These libraries and tools bills themselves as frameworks for building embedded applications

 - [rtic][rtic]
 - [Embassy][embassy]

## Marvel Cinematic Universe (aka our possible targets)

I am not here to list all producers and available chips and architectures. Mostly because I don't think that is helpful, and frankly it would take me too long to trawl through the deteriorating state of internet search to more completely map out the state of multicore microcontrollers.

What I have is a list of the chips that I have available to test right now.

 - Raspberry Pi: rp2040 and rp2350 chips have multicore support, and the `rp-rs` team has made a [multicore hal][rp-rs-hal-multi] available.
 - Espressif: ESP32 have select chips available with multiple cores and some with an asymmetric setup of a main CPU and a [low-power coprocessor.][esp32-lp-hal]
 - Nordic Semiconductor: nrf5340 has two cores. The main CPU core, called the Application Core, and a secondary core called the Network Core. Using Nordic's Connect SDK; the net-core is responsible for communicating with the radio-device. [nrf-hal][nrf-hal]

##

In which I try to list the pros and cons of multi-core processing on MCUs.



## All round example

So let's put into practice what we've been discovering by implementing a dual-core application on the Raspberry Pi RP2040.


## Conclusion and next steps
It's clear that we need better handling of multi-core computing in embedded systems.


[rp-rs-hal-multi]: tab:https://docs.rs/rp2040-hal/latest/rp2040_hal/multicore/index.html
[esp32-lp-hal]: tab:https://github.com/esp-rs/esp-hal/tree/main/esp-lp-hal
[nrf-hal]: tab:https://github.com/nrf-rs/nrf-hal
[rust-embedded-book]: tab:https://docs.rust-embedded.org/book/concurrency/index.html#multiple-cores
[droneos]: tab:https://www.drone-os.com/
[tock]: tab:https://tockos.org/
[hubris]: tab:https://github.com/oxidecomputer/hubris
[exhubris]: tab:https://github.com/cbiffle/exhubris/
[riot-os]: tab:https://doc.riot-os.org/using-rust.html
[freertos-rust]: tab:https://github.com/lobaro/FreeRTOS-rust
[rtic]: tab:https://rtic.rs/
[embassy]: tab:https://embassy.dev/

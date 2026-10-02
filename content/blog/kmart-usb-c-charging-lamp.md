+++
title = "Kmart USB-C charging lamp"
slug = "kmart-usb-c-charging-lamp"
date = "2025-04-26T12:42:00+00:00"
include_in_feeds = false
discoverable = true
is_page = false
+++

Recently I saw this YouTube video pop up claiming Kmart lied about the usb charging lamp. So I decided to get one of these lamps for myself to investigate as well as make a version of the hack myself.

What I found was very interesting and I focus of the teardown and explain what is up and why that might be.

I documented some of this already on my fediverse account.


The lamp is cheap, AU$17, but on the outside seems well put together and would fit in a nicely decorated room. It has one button and a usb-c type port. The button is a capacitive touch button and a light touch will toggle through three illumination intensities and off.

The lamp internals are accessible through the bottom where a glued on anti-slip sticker caps the bottom metallic cylinder. Inside is another plastic disc holding the electronics up. Looking in a bunch of hot melt glue holds things in place, more or less firmly. There’s a hollow, threaded metal rod going up the middle with a cable coming out for powering the LEDs at the top of the lamp. A clear plastic housing on one side has a washer and nut threaded on. It’s holding was seems to be a small pcb and the USB-C receptacle. Two wires coming out and into a heat shrink wrapped board. Another housing on the opposite side hold a single wire to the touch button. It too terminates in the main board heat shrink package. A large blue cylinder is also tucked in, haphazardly. This is the 18650 Li-ion battery. It too goes into the main board.

Liberating the components, that weren’t screwed into the wall, from their home reveals the main board to have very few components. Four JST-style connectors holding in place the 7 total wires = 2 USB, 2 BATTERY, 2 LED, 1 TOUCH. Trying to disconnect proved destructive unfortunately. First I tried the battery connector and I accidentally snapped part of the plastic. With the other three I tried cutting parts of the plastics keeping the connectors interlocked with varying degrees of success. In the end the touch wire connector housing also partially broke off. This isn’t a total disaster as the connectors still sit firmly when connected though will definitely come loose if pulled.

Next up I decided to extricate the USB pcb to see what it contained. I unscrewed the nut and got it and the washer off. The assembly then came out the front of the lamp easily. I did have to use a sharp knife to cut the sides of the soft plastic housing. So putting it back together in its original form will no longer be possible.

Looking at the board quickly revealed to me why this charging cable and the USB-C claims need to be examined carefully. It turns out the board contains a battery charge chip and a few resistors, capacitors and LEDs. Importantly for USB-C Power Delivery it doesn’t contain any CC resistors. It’s crucial for proper type c charging with PD aware charging bricks that the target device tells the power upstream of its power need. To be in spec and trigger the default power two 5.1kOhm resistors have to be added to the CC pins pulling them to ground. These are absent here. Meaning any proper charger that use usb-c to usb-c will not deliver any power. Chargers that use usb-a to output power already give the default power delivery so don’t need any extra circuitry on the target device tells. This is obviously not ideal in a world where more and more chargers are USB-C PD compliant and the reduction in price and complexity of the board would at most have saved the factory and few cents at most. It boggles the mind that they skimped on this.

To fix this oversight one can either find and alternative usb-c charge board (remember to get one that can charge the 3.7V Li-ion battery in the lamp. Or try to fix the board by trying to expose the copper on the middle two pins in the usb receptacle and solder two 5.1kOhm resistors on and then to ground.

Looking at the main board there a ubiquitous 8-pin package with some form of microcontroller. Its job is to get the touch input and then output a pwm signal to a transistor that controls the LEDs.

Now I’ve not yet implemented this, but my plan is to unsolder the mcu and bridge on wires from another board so I can implement something smarter and maybe even remote controlled.

I do have a vague plan of a strategy to get some new filament and print some bits to house a fixed usb-c charge board (the chip on the current pcb is well know and I could foresee myself designing a replacement and get a few produced). I would probably also design a compact mcu board as well and use some clever layout to fit it into the size limits at jlcpcb.

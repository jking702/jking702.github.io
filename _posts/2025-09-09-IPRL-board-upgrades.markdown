---
layout: post
title: "Upgrading power distribution"
published: false
date: 2025-09-09 20:22:30 +0100
categories: Hardware
---

I've made numerous posts on the power distribution system that I made for the Imperial Planetary Robotics Lab rover Bobcat. The end product was a practical amalgam of PCBs that provided a battery output with some of the requisit safety features. The improvements are numerous, none of which were implemented in the start due to time constraints. This is a more indepth look at two of the main improvements that are needed before the next competition. The batteries and the battery management.

# Current batteries

So the current battery to use is a 15 Ah 4S LiPo battery, which is far larger than anything you can take on a commerical airflight or get shipped. So we ended up driving the rover to try and avoid these issues. This battery worked as expected and so we need to match the similar Watt hours. We didn't use a battery management system, instead we had an ADC that measured the output voltage and warned us when we dropped below a certain voltage.

So we need a similar Wh but smaller batteries. The approach would be to then have several seperate LiPos and then put them in parallel. A big problem if you have even a minute difference between the cell voltages and a low resistance connection between them, then you have 10s of amps going between batteries completely uncontrolled. So we need something to limit the current between batteries or to hotswap out the battery being used so we can change which battery is hooked up at a given time.

## OR-ing a battery

So with regards to the current limiting approach, I saw a guy connect several Lithium cells in parallel placing a resistor between them to limit the current until the cell voltages equalised. Not a good idea, but we can do better. So we want current to flow one way into a circuit but not the other way? Well the ultimate device for that is the diode. Place a battery and connect a diode to the output of each, all going to a shared voltage bus. Now sure some issues do come about from this, imagine if you have a high current and put too much power through the diode? Okay not great. So there are specific ideal diodes that work with this. Behold the LTC4450 power OR circuit:

![Figure 1](/assets/images/OR ing circuit.png)

The only issue with this one is that this well documented circuit does not work well with high current, as in the case of IPRL they probably need something in the order of 30 A continuous and maybe 80 A peak depending on what they get up to. So instead we use a high side OR-ing controller that uses an external FET so we get to choose the rating for the FET and so we can handle anything that can be thrown at it.

We can go in another direction, such as power muxes, but this has better redundancy with no digital control pins required to maintain the operation.

# Reconcieved power board

Improving on the mistakes of the past, I made the fuck up of not having all the boards made in a single schematic. This gave me major headache when I forgot I didnt have the pins going to the correct outputs. 
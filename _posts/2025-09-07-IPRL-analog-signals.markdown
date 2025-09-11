---
layout: post
title: "pH meter analogue front end: how it works and how to integrate it"
published: false
date: 2025-09-07 13:55:30 +0100
categories: Hardware
---

During the 2025 European Rover Challenge (ERC) I worked on the power system for the Imperial Planetary Robotics Lab (IPRL) rover. Late in the cycle we were asked to add pH sensing for an astrobiology task. The quickest path was a low-cost glass pH probe held by the gripper and a small off-the-shelf amplifier board. For a production rover I’d rather bring the high-impedance analogue front end onto the arm PCB so the sensitive node stays inside the enclosure and the shielded BNC run is as short as possible; that approach is less susceptible to noise than carrying an analogue output across the harness. Timing didn’t allow that integration for ERC 2025, so this write-up is the guide I wished I’d had.

This post explains how a classic pH front end works, using DFrobot’s pH meter v1 as the conceptual reference because its schematic is publicly available, then demonstrates the key stages with SPICE (input buffering, gain/offset, filtering and calibration), and finally shares an original, from-scratch schematic and PCB that you can drop into your own designs. The design files are my work, released under an open hardware licence, and are not affiliated with DFrobot.

I’m sharing this to give future IPRL members and anyone integrating high-impedance sensors a practical, reusable reference for small-signal analogue on mixed-signal robots. If you’re building around a glass pH probe, this should help you route a clean BNC to your PCB, keep noise under control, and calibrate with confidence.

# How does the SEN0161 board work?
Yeah I wasn't too sure either, the datasheets from the glass pH probes show a nice and linear voltage output, the only issue being that it is small and so some amplification is needed.
![Figure 1](/assets/images/Ph-mv.jpg)
This is where the amplification board comes in handy, shown below.
![Figure 2](/assets/images/pH meter V1.0 SCH.pdf)

---
layout: post
title: "Upgrading Power Distribution for the IPRL Rover"
published: false
date: 2025-09-09 20:22:30 +0100
categories: Hardware
---

Designing a rover’s power system is a balancing act of performance, safety, and cost.  
For the Imperial Planetary Robotics Lab (IPRL) rover *Bobcat*, I was responsible for re-thinking the power distribution so the team can safely ship the rover to international competitions without sacrificing runtime or blowing the budget.

## Requirements and Constraints
- **Electrical load:** design target ≈20 A continuous with peaks up to ≈40 A (to be validated in testing).  
- **Battery restrictions:** airline regulations limit lithium batteries to <100 Wh for easy transport.  
- **Budget:** student competition levels—parts must be inexpensive and easy to source.  
- **Legacy system:** a single 15 Ah 4S LiPo pack feeding a custom power-splitter PCB.

The original 15 Ah pack worked electrically but exceeded airline Wh limits, forcing the team to drive the rover from London to Poland for competition.  
Shipping or flying with that pack was not feasible.

## Exploring Alternatives
We considered several options:

### Parallel Packs
Two smaller packs in parallel looked attractive on paper but posed serious safety risks.  
Any voltage difference between packs could cause large equalization currents—potentially tens of amps—leading to overheating or catastrophic failure.

### Separate Packs Per Subsystem
Powering each subsystem from its own pack would isolate the batteries but required major redesigns to add voltage monitoring and protection across multiple rails—too disruptive for the schedule.

## Final Approach: High-Side Ideal-Diode OR-ing
The winning solution was an **OR-ing circuit** that lets multiple smaller packs share a common rail while preventing cross-charging.

A simple diode OR would waste power through voltage drop and heating, so I implemented an *ideal-diode* configuration.  
The circuit uses a high-side OR-ing controller driving an external MOSFET, allowing us to select a FET with the right current rating and low Rds(on).

![Ideal Diode OR-ing Schematic](/assets/images/OR-ing-circuit.png)

Key components:
- **Controller IC:** TI **LM5050-1** high-side OR-ing controller  
- **MOSFET:** Vishay **SQJQ100E** (low Rds(on), rated well above 40 A)

Benefits:
- Airline-compliant packs can run in parallel without risk of one charging another.  
- Scales to high currents by selecting an appropriate MOSFET.  
- Minimal voltage drop and heat compared to passive diodes.

## Implementation
I designed a compact PCB “OR board” that integrates:
- The LM5050-1 controller
- A SQJQ100E MOSFET
- Wide copper pours and heavy copper layers for low resistance and heat spreading.

The board is a drop-in addition to the existing splitter with minimal wiring changes.

## Testing

Because I am no longer have access to a university lab I need to think of ways of testing this safely and cheaply. The tests I would run include:
- Continuous load testing at 20 A+ and peak testing up to 40 A
- Batteries can be swapped in and out with no signifigant issue
- Batteries that are not equally charged can be used

Firstly I need a load that can handle that much power and is also low enough resistance that it draws that amount of current. Last time I free sampled a £70 1 ohm resistor for this test, but now I think I need a new breed of dummy load. I looked at the arachnid labs Re:load Pro and it is a good breed of open source tooling. However my power consumption is far higher and so demands 500 W of dissipating power.

## Testing and Results *(planned)*
I’m currently ordering the PCB and components.  
Upcoming tests will include:
- Continuous load testing at 20 A+ and peak testing up to 40 A.  
- Measurement of voltage drop across the MOSFET and controller.  
- Thermal profiling under sustained load.  
- Verification that total battery capacity remains below the 100 Wh airline limit.

I’ll update this section with measured data, thermal images, and photos of the assembled board once testing is complete.

## Lessons Learned
This project demonstrates how careful component selection and a few grams of silicon can replace a costly redesign.  
Working within regulatory, budget, and performance constraints has required:
- Evaluating multiple architectures quickly.
- Modeling current flow and thermal limits.
- Planning a robust test procedure before hardware arrives.

---

*By integrating high-current ideal-diode OR-ing with the LM5050-1 and SQJQ100E, the IPRL team can safely transport the rover worldwide while maintaining reliable high-power operation.*

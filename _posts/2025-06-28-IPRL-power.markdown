---
layout: post
title:  "IPRL Power Distribution System"
pubished: true
date:   2025-06-28 00:35:45 +0100
categories: Hardware
---
Founded in 2023, the Imperial College Planetary Robotics Lab (IPRL) is a student run society focused on planetary robotics. Its flagship project is the development of a rover platform for the European Rover Challenge (ERC). While the rover has performed well from a mechanical standpoint, and is the only UK team to have qualified at the time of writing, it became clear that upgrades to the electrical infrastructure were necessary to support any future upgrades to the rover. For example, the use of more powerful motors for the drive, arm, or drill systems could improve mechanical performance, but would also require a more robust power solution capable of reliably delivering the higher currents demanded by such upgrades.

The following sections provide an overview of the requirements defined for this system for the 2025 competition, including the specifications with intentional overspecifications to ensure robustness, the two design iterations and the testing efforts undertaken to produce a dedicated power distribution solution.

# Requirements

The competition consists of seven tasks, each requiring the rover to operate for 10–30 minutes. To comply with the rules, the rover must be capable of running continuously for at least 60 minutes. This made it crucial to understand the current draw of each subsystem. In earlier iterations, no current draw calculations had been carried out, and the rover instead relied on two 15 Ah lead-acid batteries wired in parallel. While this configuration suggested a combined capacity of 30 Ah, [Peukert’s law](https://en.wikipedia.org/wiki/Peukert%27s_law) introduced uncertainty, as the actual usable capacity under the higher current demands of systems such as the robotic arm or drive motors was significantly lower. At the time, the subsystems were disassembled, making direct current measurements impossible, so the required battery size had to be estimated. Initial calculations were based on the motors used for the arm, drive, and drill subsystems, along with comparisons to similarly sized electrical products. These comparisons, ranging from golf carts to ride-on lawnmowers, suggested that a battery capacity of 12–15 Ah would be sufficient. Some resourceful scavenging at a local battery disposal point turned up a 15 Ah lithium polymer (LiPo) battery, saving between £100 and £300 of the project budget. Stall currents for the drive motors were around 8 A each, meaning the system needed to provide up to 32 A in scenarios such as turning on the spot with the differential drive system, where all four motors can briefly stall simultaneously.

To meet safety standards, the competition requires a hardware emergency stop button, along with indicator lights to display the rover’s current state and fuses to protect components from overcurrent damage.

Although not mandatory, additional systems such as current monitoring and battery monitoring would greatly improve reliability. Current monitoring would provide valuable diagnostic data in the event of a major failure, while battery monitoring became essential with the use of a LiPo battery to ensure the voltage remained within the safe operating range.

As always, IPRL is a heavily underfunded project, operating on budgets typically only 1–10% of those available to a standard qualifying rover at the ERC. As a result, cost-effectiveness is one of the most critical requirements in the design of our rover.

In summary, the initial power system requirements for the rover were:
- Ability to power the rover for 60 minutes continuously
- Ability to supply 30 A to a given subsystem
- Battery voltage monitoring
- Output current monitoring
- Hardware emergency stop button
- Indicator lights
- Fused subsystems for overcurrent protection
- Cost-effectiveness

With the power system requirements defined, the next step was to design a way of distributing that power safely and reliably to all the rover’s subsystems. A simple wiring harness might have worked for a small prototype, but for a competition rover it would have quickly become messy, difficult to debug, and prone to failure under stress. To avoid this, I decided to design a dedicated power distribution PCB. The first iteration of this board brought together the basic elements of the system, including fuses and an emergency stop, but as with many first attempts, it exposed a number of weaknesses that needed to be addressed in the next design.

# First iteration
The first power distribution PCB was built around an ESP32-S3 microcontroller. This was chosen less for necessity than for accessibility: the juniors could program it easily in the Arduino IDE, and its processing power and flexibility meant it could grow with the system. Since the onboard ADC of the ESP32 is unreliable, an ADS1115 external ADC was included for battery voltage monitoring.

For current monitoring, ACS712 Hall-effect sensors were used. These were already familiar from the drive system and, when ordered in bulk, were far cheaper than building a shunt resistor solution. Their 30 A range was sufficient, and although they required a 5 V supply, the accuracy was more than enough for the rover’s needs. Importantly, they reliably detected current spikes without introducing significant noise into the system.

Battery management was handled simply, using a voltage divider feeding the ADC. This gave accurate enough readings to ensure the LiPo battery remained within safe limits, and charging was left to external equipment: a Bat-Safe enclosure and a B6AC v2 balance charger. To save both cost and weight, the battery voltage was not bucked down to 12 V. High-current buck converters can be expensive, inefficient, and heavy, and the rover also had a strict mass budget. Instead, subsystems were designed to tolerate the native voltage of the 4S LiPo pack. This approach was less refined than a regulated bus, but it significantly reduced system complexity and cost.

The emergency stop system needed more creativity. Commercial e-stop buttons are rarely rated beyond 1 A, which made them unsuitable for the rover. To get around this, an automotive relay rated for 80 A was used, with the e-stop switch controlling the relay coil. This worked, but one flaw emerged: the relay was designed for 12 V systems, while the rover’s 4S LiPo sat closer to 16 V. The higher coil voltage caused heating, which was later solved in the second iteration by adding a current-limiting resistor. Despite that, the e-stop functioned reliably throughout testing.

Indicator lights were implemented with a simple but flexible scheme. A red LED was wired directly to show battery connection and fuse health, while amber and green indicators were GPIO-controlled, leaving their exact meanings open for future software mapping.

The PCB itself was designed as a four-layer stack-up. The top two layers acted as solid copper planes for power distribution, with stitching vias reinforcing areas of higher current density. The third layer was dedicated to routing analog signals, and the fourth served as a ground return path. This design was conservative: there were no overheating issues, but the heavy copper pours and large board size made manufacturing expensive. The board had to be oversized to accommodate an XT60 output for each subsystem, plus a screw terminal, and this drove costs up disproportionately.

While Iteration 1 met the basic functional goals of distributing power, adding monitoring, and integrating safety features, it was overbuilt, oversized, and expensive. These lessons directly shaped the second iteration, where the focus shifted toward tighter cost control, more compact design, and more efficient power handling.



# Second iteration

The final redesign was motivated by the limited number of ports on the original power board, which only featured two. To address this, the power distribution system was divided into a main board and four identical secondary boards. Each secondary board acted as a power splitter, receiving power from the main board and providing multiple ports to supply an individual subsystem. This modular arrangement also reduced manufacturing costs: PCB pricing from JLCPCB increases sharply for board sizes larger than 100 mm × 100 mm, so producing several smaller boards was cheaper than fabricating one large one.

The main design challenge with this split architecture was ensuring it could still safely supply up to 30 A. This requirement provided a comfortable safety margin for current needs and allowed capacity for potential future upgrades.

## Connector Selection

To connect the main and secondary boards, a connector was required that was inexpensive, widely available, capable of carrying high currents, and able to transmit analogue signals for the inductive current measurement ICs on each secondary board. The AMASS XT60 2+4 connector was initially considered, but it lacked a reference design, standard PCB footprint, and datasheet, making integration impractical.

Instead, a more novel solution was adopted by repurposing PCIe lane connectors commonly used in PCs. A PCIe lane typically assigns nine pins near the key for power and the remaining pins for signals. In this design, the roles were inverted: the signal pins were used to distribute power, while the power pins carried the analogue measurement signals. Although unconventional, this solution offered several advantages. PCIe connectors are extremely low-cost (under £1 each), widely available, and well-documented, with standard footprints and CAD models, which significantly reduced design uncertainty. According to IPC-2221 calculations, this arrangement could handle up to 70 A, comfortably surpassing the 30 A target. However, it should be noted that IPC-2221 data is only validated up to 35 A, so the 70 A figure is an extrapolation rather than a certified limit.

The primary drawback of using PCIe connectors is mechanical reliability under vibration, since typical PC applications are not exposed to movement. To mitigate this, plated mounting holes were added to the secondary boards so they could be securely fastened inside the power system enclosure. Figures 1–3 show the complete structure of the main and secondary boards, along with how they interconnect via PCIe lanes.

![Figure 1](/assets/images/board_exploded.png)
**Figure 1:** Exploded view showing how the secondary boards connect to the main board via PCIe connectors.  

![Figure 2](/assets/images/main_board.png)
**Figure 2:** Layout of the main power distribution board, highlighting the PCIe connectors and copper pours.  

![Figure 3](/assets/images/secondary_board.png)
**Figure 3:** Standardised secondary power board used to supply individual subsystems.  

## High-Current Copper Pours

To ensure current capacity, the main power board used extensive copper pours on both the top and bottom layers, as shown in Figure 4. The narrowest section of the primary current-carrying path measured approximately 22.6 mm. IPC-2221 calculations indicated this width would result in only a 20 °C temperature rise at 31 A continuous current, which comfortably maintains the 30 A safety margin and eliminates the need for heatsinks.

It should be noted that IPC-2221 calculations are based on isolated single traces and do not account for heat conduction into surrounding copper areas. Since this design used large, continuous copper fills without thermal reliefs, the board can likely handle more than 30 A. This approach does make soldering more difficult, as the copper planes quickly draw heat away from through-holes, but it helps spread thermal load across the board, reducing the need for external heatsinking.

![Figure 4](/assets/images/copper_pours.png)
**Figure 4:** Top copper pour highlighted on the main power board, designed to safely carry high currents.  

## Addressing Earlier Issues

The redesign also allowed two issues from Iteration 1 to be resolved.

### Relay Heating in the E-stop Circuit
The automotive relay used for the emergency stop overheated in the first design because the 14.8 V battery voltage drove significantly more current through a coil designed for 12 V. Using the maximum battery voltage of 16.8 V and assuming constant coil resistance, the power dissipated increased by 96%, adding roughly 1 W of heat. This raised concerns about long-term reliability. Several solutions were considered, including keeping the relay closed by default to reduce coil duty time. Ultimately, a simpler approach was chosen: a series shunt resistor was added to safely dissipate the excess power, as shown in the revised schematic in Figure 5.

![Figure 5](/assets/images/relay_schematic.png)
**Figure 5:** Left schematic is the original relay implementation and the right is the revised implementation with a shunt resistor.  

### LED Indicator Driver Circuit
In the initial design, an N-channel MOSFET was incorrectly used for low-side switching of the indicator lights. This assumed the lights had isolated grounds, but in reality they shared a common ground and isolated inputs, meaning only high-side switching would work. Additionally, the MOSFET gate was tied directly to an ESP32 GPIO, which could never pull the gate high enough for proper operation. The redesign corrected this with a two-MOSFET high-side switch and a pull-up resistor, ensuring reliable control. An extra LED driver was also added to support additional indicators in the future.

![Figure 6](/assets/images/indicator_schematic.png)
**Figure 6:** Left schematic is N-channel MOSFET low sided switch right figure is the corrected high-side switching circuit for the indicator lights.  

# Conclusion
With two design iterations, the power distribution system matured from an oversized, overbuilt prototype into a modular, reliable, and cost-effective solution. The final version not only met the competition requirements for safety and runtime but also stayed within the razor-thin budget of the project. The process highlighted the constant trade-offs between cost, safety, and performance, and showed how unconventional solutions like PCIe connectors could deliver real advantages when used creatively.

With the PCB complete, the next major task was designing and manufacturing a container to protect it and integrate it into the rover. That part of the build will be covered in the next post.
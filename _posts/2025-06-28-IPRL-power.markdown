---
layout: post
title:  "Imperial College Robotics Society Power Distribution"
date:   2025-06-28 00:35:45 +0100
categories: Hardware
---
Founded in 2023, the Imperial College Planetary Robotics Lab (IPRL) is a student run society focused on planetary robotics. Its flagship project is the development of a rover platform for the European Rover Challenge (ERC). While the rover has performed well from a mechanical standpoint, and is the only UK team to have qualified at the time of writing, it became clear that upgrades to the electrical infrastructure were necessary to support any future upgrades to the rover. For example, the use of more powerful motors for the drive, arm, or drill systems could improve mechanical performance, but would also require a more robust power solution capable of reliably delivering the higher currents demanded by such upgrades.

The following sections provide an overview of the requirements defined for this system for the 2025 competition, including the specifications with intentional overspecifications to ensure robustness, the two design iterations and the testing efforts undertaken to produce a dedicated power distribution solution.

# Specifications
- Cheap
- Able to power the rover for 60 minutes continuous
- 

# Second iteration

The final redesign was motivated by the limited number of ports on the original power board, which only featured two. To address this, the power distribution system was divided into a main board and four identical secondary boards. Each secondary board acts as a power splitter, receiving power from the main board and providing multiple ports to supply an individual subsystem. This modular arrangement had the benefit of reducing manufacturing costs since PCB pricing from JLCPCB rises sharply for board sizes exceeding 100 mm × 100 mm, and so buying several separate boards was cheaper than buying several larger boards. The primary concern with adopting this split design was ensuring it could still safely supply up to 30 A, a requirement set to provide a comfortable safety margin for current needs and to accommodate future upgrades that might demand higher power.

To connect the main and secondary boards and support the 30 A safety margin, a connector was required that was inexpensive, widely available, capable of carrying high currents, and able to transmit analogue signals for the inductive current measurement ICs on each secondary board. The AMASS XT60 2+4 connector, which combines power and signal pins, was initially considered. However, it lacked a reference design, standard PCB footprint, or even a datasheet to create one, making integration difficult. Instead, a more novel solution was adopted by repurposing PCIe lane connectors commonly used in PCs. A PCIe lane in a PC typically assigns 9 pins on the left side of the mechanical key for power and the remaining 155 pins for signals, this design inverted the roles: the signal pins were used to distribute power, while the power pins carried the analogue measurement signals. Although unconventional, this choice offers several advantages. PCIe connectors are extremely low cost at under £1 each, widely available, and standard in industry and so have standard footprints and CAD models, which significantly reduced design uncertainty. Moreover, this configuration is expected to handle up to 70 A, comfortably surpassing the safety margin requirements. It should be noted, however, that the IPC-2221 current handling calculations used to estimate this capability are empirical models that have only been validated up to 35 A, so the 70 A figure is an extrapolation rather than a certified limit. Nevertheless, it remains comfortably above the design target. The primary drawback of using PCIe connectors is the mechanical reliability under vibrations as the typical application does not see a PC moving or being subject to vibrations. To mitigate this, plated mounting holes were added to the secondary boards so they could be securely fastened to the power system enclosure. The complete structure of the main board and secondary boards, including how they interconnect via the PCIe lanes, is shown in Figures 1, 2, and 3.

**Figure 1:** Exploded view showing how the secondary boards connect to the main board via PCIe connectors.  
![Figure 1](Final%20Design/Images/Electronics/board_exploded.png)

**Figure 2:** Layout of the main power distribution board, highlighting the PCIe connectors and copper pours.  
![Figure 2](/assets/images/main_board.png)

**Figure 3:** Standardised secondary power board used to supply individual subsystems.  
![Figure 3](Final%20Design/Images/Electronics/secondary_board.png)

Once this configuration was established, the physical implementation took advantage of extensive copper pours on both the top and bottom layers of the main power board to reliably carry high currents, as shown in Figure 4. This figure highlights the primary current-carrying path, with a measuring tape indicating the width of its narrowest section, measured at approximately 22.6 mm. According to IPC-2221 calculations, this width would result in only a 20 °C temperature rise when continuously carrying 31 A, comfortably maintaining the 30 A safety margin and eliminating the need for heatsinks. It should be noted that IPC-2221 calculations are based on isolated single traces and do not account for heat conduction into surrounding copper areas. Since this design extensively uses copper planes designed as complete fills without thermal reliefs to maximise heat conduction, the board can likely handle currents greater than 30 A. Although this approach makes soldering more challenging because heat is more readily drawn away from through-holes, it helps spread heat into the surrounding copper, further reducing the need for external heatsinking.

**Figure 4:** Top copper pour highlighted on the main power board, designed to safely carry high currents.  
![Figure 4](Final%20Design/Images/Electronics/copper_pours.png)

The redesign also allowed two other issues with the power system to be addressed. The first issue was excessive heating in the automotive relay used as part of the Estop circuit. This was caused by the 14.8 V battery voltage dissipating more power than expected through a relay designed for 12 V. Calculations using the maximum voltage from the battery of 16.8 V as the input voltage and assuming a constant coil resistance then using P = V²/R showed that a 40% increase in voltage increased the power consumption in the coil by 96% compared to the expected 12 V. This amounted to an extra 1 W of power which led to heating of the coil, which was thought could cause long term damage to the relay. Several approaches were considered to reduce heating, such as always keeping the relay closed and only opening it via the closing contacts so power would be dissipated in the coil only when the power system was switched off. A simpler solution was to add a series shunt resistor to safely dissipate this additional 1 W of power, as shown in the revised schematic in Figure 5.

**Figure 5:** Left schematic is the original relay implementation and the right is the revised implementation with a shunt resistor.  
![Figure 5](Final%20Design/Images/Electronics/relay_schematic.png)

The second issue was with the LED driver switch originally designed to control the indicator lights. In the initial circuit, an N-channel MOSFET was incorrectly used under the assumption that the indicator lights had individually isolated grounds, without realising that the power inputs were individually isolated and all lights shared a common ground therefore only a high-side switch would work. Additionally, the gate was connected directly to a GPIO of the ESP32, which would never be able to pull the gate high enough to turn the MOSFET on. A complete redesign corrected this by using a two MOSFET high-side switch with a pull-up resistor, as shown in Figure 6. An additional LED switch was added in the event that another indicator light was used in the future or that the current configuration was changed to have all indicator light be individually controllable.

**Figure 6:** Left schematic is N-channel MOSFET low sided switch right figure is the corrected high-side switching circuit for the indicator lights.  
![Figure 6](Final%20Design/Images/Electronics/indicator_schematic.png)

---
layout: post
title:  "Designing a power enclosure"
date:   2025-08-13 11:37:45 +0100
categories: Hardware
---
A usually overlooked part of electronics is containing the PCB or hardware after it has been made. This is a look into the iterative process of a battery/ power distribution box for a rover that competed in the European Rover Challenge (ERC) 2025 for the Imperial Planetary Robotics Lab (IPRL) team.

## Specification
Broadly speaking it is a box that has to be able to protect a battery and the power splitter PCB from rain and external damage, therefore it has to be able to:
- Fit all the components, PCB main and secondary, battery and relay
- Fit all cables to be routed within the enclosure (relay cables, battery)
- Cutouts for XT60 outputs and Estop button

Given that the battery will need to be swapped out during the competition, it would be nice if it could remove that easily, if it blows a fuse in the board we have to be able to replace those quickly.
- Easy removable battery
- Quick access to fuses for replacement

Finally the restrictions on manufacturing (i.e this was made with a chronically underfunded student society) and therefore we have to keep to the manufacturing limitations of a student run maker space so that means we can only use:
- Laser cutting with acrylic
- FDM printers loaded with PLA or PETG
- Screws
- Heat set inserts

With the requirements and restrictions in mind, here are the prototypes that insued.

# Similar Designs
## V1
It's a fucking box, with all that would be needed. Carelessly made into one 3D print. But we can do better than this.

- I need to use a divot in the plastic to shield the cables, this thing does not need to be waterproof but it does need to not short in outdoor conditions
  - this means using something like either of these designs
    - https://www.thingiverse.com/thing:1264391
    - https://www.thingiverse.com/thing:6392562
- the only issues with that is that the length of the box would mean there is too much wasted space in this clam like box, also just not that great.

## V2
To still fit these requirements, a handle on top of the cover along with the 

### Base
### Frame with PCB support
### Secured frame
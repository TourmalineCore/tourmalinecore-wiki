# Preparing Design Layouts for Development #

This document describes the requirements for design layouts before they are handed over for development. The goal is to reduce the number of questions from developers, speed up development, and reduce the number of bugs.

## 1. Figma File Structure ##


The Figma file should be clear to any developer or designer opening it for the first time. 
The recommended page structure is: 

![](images/structure.png)

__Recommendations:__
- Screens are grouped by sections
- No chaotic frames
- Archive is placed separately
- Components are stored in a centralized way

## 2. Responsive Design ##

All layouts must be prepared for the main screen sizes: Desktop, Laptop, Tablet, and Mobile, based on popular user devices and the screen widths at which the interface starts to rearrange.

![](images/breakpoints.png)

For each breakpoint, set specific font sizes, paddings, and component dimensions, while leaving the names of the text styles the same.

Blocks and components should logically rearrange themselves when the screen size changes. For example, horizontally arranged cards on Desktop should stack vertically on mobile devices, preserving order and meaning (exceptions are possible).


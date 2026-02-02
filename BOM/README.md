# BOM files & Gerber changelog


Updated the gerber to v1.1. No drastic changes for anyone that has v1.0 already. Removed the 10k resistors from the controls (if you have v1.0, you can just leave them unpopulated, thats what I did), after extensive testing it wasn't required (they were added incase the PCAL's internal pull ups were not enough for a large board) Tidied up some routing and also angled the UART port slightly to allow a 90 degree pinheader. Will keep the BOM for v1.0 for now.

Small changes for v1.11, components and placements unchanged so BOM's v1.1 are still valid. Gerber updated to v1.11, small QOL changes: Increased via size on power rails (as many as I could due to space restrictions in some areas but these are ones at the end, all early and main rails are increased) to reduce chances of power issues, not that I've had any issues myself. Increased the track size on the LCD Clock line as it does transmit quite a lot of data so chunkier the better, again never had any issues. A few traces moved slightly to accomodate both changes but nothing drastic.

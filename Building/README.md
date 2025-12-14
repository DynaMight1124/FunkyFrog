\# FunkyFrog OS Compilation Guide



This guide documents the process of compiling the FunkyFrog OS image. It is based on the original \[FunKey-OS](https://github.com/DrUm78/FunKey-OS) but includes specific modifications for the larger 320x240 display and custom hardware adjustments.



\*\*Note:\*\* This guide uses a direct modification method (replacing files in the build directory) rather than a full clean environment setup. This is often faster given the complex dependency downloads required by the main OS.



\## Prerequisites



&nbsp; \* \*\*OS:\*\* Ubuntu 22.04.5 LTS (I used WSL2 on Windows 11).

&nbsp; \* \*\*Base Build:\*\* Follow the \[official FunKey-OS build guide](https://github.com/DrUm78/FunKey-OS?tab=readme-ov-file#build-on-a-physicalvirtual-machine).

&nbsp;     \* \*Estimated time:\* \\~90 minutes.

&nbsp;     \* \*\*Important:\*\* Ensure the base FunKey OS compiles successfully \*before\* applying the FunkyFrog adjustments below.



-----



\## 1\\. Linux Kernel \& Drivers



The first step is updating the video drivers and device tree to support the 320x240 screen. We need to apply these changes to both the \*\*FunKey\*\* (main OS) and \*\*Recovery\*\* builds.



\### Video Driver (fbtft)



We need to replace the driver files and force a recompile.



1\.  \*\*Locate the directories:\*\*

&nbsp;     \* `FunKey-OS/Recovery/output/build/linux-custom/drivers/staging/fbtft/`

&nbsp;     \* `FunKey-OS/FunKey/output/build/linux-custom/drivers/staging/fbtft/`

2\.  \*\*Action:\*\*

&nbsp;     \* Copy the custom driver files (from fbtft-drivers.zip) into \*both\* directories, overwriting existing files.

&nbsp;     \* \*\*Crucial:\*\* Delete any existing `.o` (object) files in these folders to ensure they are recompiled.



\### Device Tree (DTS)



This controls the hardware configuration.



1\.  \*\*Locate the file:\*\* `sun8i-v3s-funkey.dts`

&nbsp;     \* Path 1: `FunKey-OS/FunKey/output/build/linux-custom/arch/arm/boot/dts/`

&nbsp;     \* Path 2: `FunKey-OS/Recovery/output/build/linux-custom/arch/arm/boot/dts/`

2\.  \*\*Action:\*\* Overwrite the file in both locations.



\### Boot Logos (Optional)



The original logos work fine, but you can update them if desired.



1\.  \*\*Locate the directory:\*\* `FunKey\\output\\build\\linux-custom\\drivers\\video\\logo \& Recovery\\output\\build\\linux-custom\\drivers\\video\\logo`.

2\.  \*\*Action:\*\*

&nbsp;     \* Overwrite `logo\_funkey\_clut224.ppm` and `logo\_funkeyrecovery\_clut224.ppm`.

&nbsp;     \* Delete any `.o` and `.c` files in the folder to force a rebuild.



> \*\*Tip:\*\* To create a custom logo, use a 320x240 PNG (limited colors/B\\\&W recommended) and run:

> `pngtopnm boot.png | ppmquant 224 | pnmnoraw > logo\_funkey\_clut224.ppm`

> \*(Requires `netpbm`: `sudo apt-get install netpbm`)\*



\### Recompile Linux



Run the following commands to apply the kernel changes:



```bash

make FunKey/linux-rebuild

make Recovery/linux-rebuild

```



-----



\## 2\\. Menu Resources



The default background image is 240x240, which causes the menu to render off-center on the wider screen.



&nbsp; \* \*\*File:\*\* `zone\_bg.png`

&nbsp; \* \*\*Location:\*\* `FunKey/board/funkey/rootfs-overlay/usr/games/menu\_resources/`

&nbsp; \* \*\*Action:\*\* Replace with the 320x240 version.



\*Note: Third-party OPKs compiled for 240x240 will still work but may appear off-center.\*



-----



\## 3\\. RetroFE



Update RetroFE to utilize the full horizontal width.



\### Code Changes



1\.  \*\*Edit File:\*\* `FunKey/output/build/retrofe-RetroFE-FunKey-1.1.4/RetroFE/Source/Menu/MenuMode.cpp`



&nbsp;     \* \*\*Find:\*\* `#define SCREEN\_HORIZONTAL\_SIZE`

&nbsp;     \* \*\*Change to:\*\*

&nbsp;       ```cpp

&nbsp;       #define SCREEN\_HORIZONTAL\_SIZE       320 //RES\_HW\_SCREEN\_HORIZONTAL

&nbsp;       ```



2\.  \*\*Edit Config:\*\* `FunKey-OS/FunKey/board/funkey/rootfs-overlay/usr/games/settings.conf`



&nbsp;     \* \*\*Set:\*\* `horizontal = 320`



\### Themes



The default themes will stretch awkwardly.



1\.  \*\*Navigate to:\*\* `FunKey-OS/FunKey/board/funkey/rootfs-overlay/usr/games/layouts`

2\.  \*\*Action:\*\* Delete all contents and replace with the contents of `320Themes.zip`.

3\.  \*\*Cleanup:\*\* Also delete all files in `FunKey-OS/FunKey/output/target/usr/games/layouts` to ensure the old themes are removed during the next compile.



-----



\## 4\\. PicoArch 



Recent official PicoArch updates have scaling issues on 320x240 screens. We must revert to a specific working version.



\### Setup



1\.  \*\*Download:\*\* Use the \[FunkyFrog branch](https://github.com/DynaMight1124/picoarch/tree/FunkyFrog) or `picoarch-working.zip`.

2\.  \*\*Location:\*\* `FunKey-OS/FunKey/output/build/picoarch-HEAD`

3\.  \*\*Action:\*\* Overwrite the directory contents.



\### Manual Code Changes (If not using the zip)



&nbsp; \* \*\*File:\*\* `scale.h`

&nbsp;   ```c

&nbsp;   #ifdef FUNKEY\_S

&nbsp;   #define SCREEN\_WIDTH 320

&nbsp;   #else

&nbsp;   #define SCREEN\_WIDTH 320

&nbsp;   #endif

&nbsp;   ```

&nbsp; \* \*\*File:\*\* `funkey/fk\_menu.c`

&nbsp;   ```c

&nbsp;   #define SCREEN\_HORIZONTAL\_SIZE       320 //RES\_HW\_SCREEN\_HORIZONTAL

&nbsp;   ```



\### Recompile \& Add Cores



1\.  \*\*Compile:\*\*

&nbsp;   ```bash

&nbsp;   make FunKey/picoarch-rebuild

&nbsp;   ```

2\.  \*\*Add Cores:\*\* The compile process does not build the emulator cores.

&nbsp;     \* Extract `Picoarch-cores.zip` to: `FunKey-OS/FunKey/output/target/usr/games/`



> \*\*Troubleshooting:\*\* If you encounter build errors when switching versions:

>

> 1.  `make FunKey/picoarch-dirclean`

> 2.  `make FunKey/picoarch-rebuild` (This pulls the official repo).

> 3.  Re-copy your modified files.

> 4.  `make FunKey/picoarch-rebuild` again.



-----



\## 5\\. GMenu2X



Update the resolution definitions and theme structure.



\### Code Changes



1\.  \*\*File:\*\* `FunKey/package/gmenu2x/src/gmenu2x.cpp`

&nbsp;     \* \*\*Change:\*\* `#define SCREEN\_WIDTH  320`

2\.  \*\*File:\*\* `FunKey/package/gmenu2x/src/funkeymenu.cpp`

&nbsp;     \* \*\*Change:\*\* `#define SCREEN\_HORIZONTAL\_SIZE       320 //RES\_HW\_SCREEN\_HORIZONTAL`



\### Theme Directory



GMenu2X searches for themes based on resolution.



1\.  \*\*Navigate to:\*\* `FunKey/output/build/gmenu2x-HEAD/data/skins`

2\.  \*\*Action:\*\* Create a folder named `320x240`. Copy all contents from the `240x240` folder into this new folder.



\### Recompile



```bash

make FunKey/gmenu2x-rebuild

```



-----



\## 6\\. Build the Image



Once all changes are applied, run:



```bash

make

```



The output files (`.img` and `.fwu`) will be generated in the `images` directory.



-----



\## Troubleshooting



\*\*Partition Size Error:\*\*

If `make` fails because the image size is too big or too small:



1\.  Open `genimage.cfg` in the main directory.

2\.  Locate `partition FunKey`.

3\.  Adjust the size (e.g., increase from default `250M` to `260M` or `500M`).


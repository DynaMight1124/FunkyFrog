# FunkyFrog OS Compilation Guide

This guide documents the process of compiling the FunkyFrog OS image. It is based on the FunKeyOS by [DrUm78](https://github.com/DrUm78/FunKey-OS) but includes specific modifications for the larger 320x240 display and custom hardware adjustments as well as some cosmetic changes due to the change of resolution/aspect ratio.

**Note:** This guide uses a direct modification method (replacing files in the build directory) rather than a full clean environment setup. This is not the proper way to do things, its a little hacky but is quicker to setup for such a niche device.

## Prerequisites

  * **OS:** Ubuntu 22.04.5 LTS (I used Windows 11 running WSL2)
  * **Base Build:** Follow the [official FunKey-OS build guide](https://github.com/DrUm78/FunKey-OS?tab=readme-ov-file#build-on-a-physicalvirtual-machine).
      * *Estimated time:* \~90 minutes.
      * **Important:** Ensure the base FunKey OS compiles successfully *before* applying the FunkyFrog adjustments below.

-----

## 1\. Linux Kernel & Drivers

The first step is updating the video drivers and device tree to support the 320x240 screen. We need to apply these changes to both the **FunKey** (main OS) and **Recovery** builds.

### Video Driver (fbtft)

We need to replace the two driver files and force a recompile.

1.  **Locate the directories:**
      * `FunKey-OS/Recovery/output/build/linux-custom/drivers/staging/fbtft/`
      * `FunKey-OS/FunKey/output/build/linux-custom/drivers/staging/fbtft/`
2.  **Action:**
      * Copy the two driver files (from fbtft-drivers.zip) into *both* directories, overwriting existing files.
      * **Crucial:** Delete any existing `.o` (object) files in these folders to ensure they are recompiled.

### Device Tree (DTS)

This controls the hardware configuration.

1.  **Locate the file:** `sun8i-v3s-funkey.dts`
      * Path 1: `FunKey-OS/FunKey/output/build/linux-custom/arch/arm/boot/dts/`
      * Path 2: `FunKey-OS/Recovery/output/build/linux-custom/arch/arm/boot/dts/`
2.  **Action:** Overwrite the file in both locations.
   
**Tip:** You can adjust the charge rate within the DTS file. Look for constant-charge-current-max-microamp. 500000 is 500mAh, 1000000 is 1000mAh. The chip supports a maximum of 1500000 (1500mAh).

### Boot Logos (Optional)

The original logos work fine, but you can change them if desired.

1.  **Locate the directories:**
      * `FunKey-OS\FunKey\output\build\linux-custom\drivers\video\logo`
      * `FunKey-OS\Recovery\output\build\linux-custom\drivers\video\logo` 
2.  **Action:**
      * Overwrite `logo_funkey_clut224.ppm` and `logo_funkeyrecovery_clut224.ppm`.
      * Delete `.o` and `.c` files related to logo_funkey in the folder to force a rebuild.

> **Tip:** To create a custom logo, use a 320x240 PNG (limited colors/B\&W recommended) called boot.png and run:
> `pngtopnm boot.png | ppmquant 224 | pnmnoraw > logo_funkey_clut224.ppm`
> *(Requires `netpbm`: `sudo apt-get install netpbm`)*

### Recompile Linux

Run the following commands to apply the kernel changes:

```bash
make FunKey/linux-rebuild
make Recovery/linux-rebuild
```

-----

## 2\. Menu Resources

The default menu image is 240x240, which causes the menu to render off-center on the wider screen.

  * **File:** `zone_bg.png`
  * **Location:** `FunKey/board/funkey/rootfs-overlay/usr/games/menu_resources/`
  * **Action:** Replace with the 320x240 version, you can also backup the original if you want.

*Note: Third-party OPKs compiled for 240x240 will still work but may appear off-center.*

-----

## 3\. RetroFE 

Update RetroFE to utilize the full horizontal width.

### Code Changes

1.  **Edit File:** `FunKey/output/build/retrofe-RetroFE-FunKey-1.1.4/RetroFE/Source/Menu/MenuMode.cpp`

      * **Find:** `#define SCREEN_HORIZONTAL_SIZE`
      * **Change to:**
        ```cpp
        #define SCREEN_HORIZONTAL_SIZE       320 //RES_HW_SCREEN_HORIZONTAL
        ```

2.  **Edit Config:** `FunKey-OS/FunKey/board/funkey/rootfs-overlay/usr/games/settings.conf`

      * **Set:** `horizontal = 320`

Run the following command to recompile:

```bash
make FunKey/retrofe-rebuild
```

### Themes

Themes designed for FunKey will work ok but stretch to the full screen so dont look great, however I have amended some themes for 320x240. They are not perfect as most had the backgrounds stretched Paint rather than recreated at the proper resolution but they do look better than the original ones designed for 240x240 as I also edited the xml files to keep other aspects of the theme from stretching (such as logos, boxarts etc).

1.  **Navigate to:** `FunKey-OS/FunKey/board/funkey/rootfs-overlay/usr/games/layouts`
2.  **Action:** Delete all contents and replace with the contents of `320Themes.zip`.
3.  **Cleanup:** Also delete all files in `FunKey-OS/FunKey/output/target/usr/games/layouts` to ensure the old themes are removed during the next compile.

-----

## 4\. PicoArch

The official PicoArch releases have scaling issues on 320x240 screens. We must use to a specific working version.

### Setup

1.  **Download:** Use the [FunkyFrog branch](https://github.com/DynaMight1124/picoarch/tree/FunkyFrog) or `picoarch-working.zip`.
2.  **Location:** `FunKey-OS/FunKey/output/build/picoarch-HEAD`
3.  **Action:** Overwrite the directory contents.
* **Crucial:** Delete any existing `.o` (object) files in these folders to ensure they are recompiled.


### Recompile & Add Cores

1.  **Compile:**
    ```bash
    make FunKey/picoarch-rebuild
    ```
2.  **Add Cores:** The compile process builds PicoArch but does not build the emulator cores. Its possible that future updates may change to use different cores for systems or add additional systems so keep this in mind. Also note that these cores do not need to be compiled to 320x240, as long as PicoArch is compiled for 320x240 then cores compiled for FunKey will work correctly.
      * Extract `Picoarch-cores.zip` to: `FunKey-OS/FunKey/output/target/usr/games/` Note theres also a folder called 'port-cores' these need to go into: `FunKey-OS\FunKey\output\target\usr\local\share\OPKs\Libretro` and Duke3D needs to go into the 'Native' folder, matching the .png file name.

> **Troubleshooting:** If you encounter build errors when switching versions:
>
> 1.  `make FunKey/picoarch-dirclean`
> 2.  `make FunKey/picoarch-rebuild` (This pulls the official repo).
> 3.  Re-copy your modified files.
> 4.  `make FunKey/picoarch-rebuild` again.

-----

## 5\. GMenu2X

Update the resolution definitions and theme structure.

### Code Changes

1.  **File:** `FunKey/output/build/gmenu2x-HEAD/src/gmenu2x.cpp`
      * **Change:** `#define SCREEN_WIDTH  320`
2.  **File:** `FunKey/output/build/gmenu2x-HEAD/src/funkeymenu.cpp`
      * **Change:** `#define SCREEN_HORIZONTAL_SIZE       320 //RES_HW_SCREEN_HORIZONTAL`

### Theme Directory

GMenu2X searches for themes based on resolution.

1.  **Navigate to:** `FunKey/output/build/gmenu2x-HEAD/data/skins`
2.  **Action:** Create a folder named `320x240`. Copy all contents from the `240x240` folder into this new folder. Themes could maybe use slight adjustment to 320x240 however the 240x240 themes look mostly fine imo.

### Recompile

```bash
make FunKey/gmenu2x-rebuild
```

-----

## 6\. Build the Image

Once all changes are applied, run:

```bash
make
```

The output files (`.img` and `.fwu`) will be generated in the `images` directory.

-----

## Troubleshooting

**Partition Size Error:**
If `make` fails because the image size is too big or too small:

1.  Open `genimage.cfg` in the main directory.
2.  Locate `partition FunKey`.
3.  Adjust 'size' (e.g. default is `350M` so adjust to 360M, you may need to play around with this figure to get it to compile).

**Any kind of issue after a successful compile:**
e.g. garbled screen, emulators not 320x240 etc. Generally I have found if I start from a fresh base, copy across the files in this guide and then have issues, its usually related to .o files not being recompiled after the change. I would recommend re-visiting the files replaced/amended, deleting their .o files and recompiling.


Also note this guide is based on the current FunkeyOS repo as of 4th March 2026, its very possible that future updates make changes which will require adjustment to the above. When setting up the build environment it will always pull the latest packages/updates at the time, just take this into consideration if errors occur. I doubt anyone will actually build a FunkyFrog let alone setup a build environment for it but if I dont write it down now, I'll forget in a few months myself!

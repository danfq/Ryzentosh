# GPU, DRM & PAT (Page Attribute Table)

## GPU

### The Issue
The `RX580 2048SP` Card is a chinese version of the well-known `RX580 Series` Card, made by AMD.

Since it isn't exactly a "proper" version of the `RX580` Card, MacOS won't recognize it, and you'll end up with a bootloop because of it.

To fix that, we must flash a new `VBIOS` Firmware on the Card.

In other words, we're going to make it <i>think</i> it's an `RX570 Series` Card.

### Necessary Tools

To begin, all you need is the `AMDVBFlash` tool, which you can download [here](https://www.techpowerup.com/download/ati-atiflash/), and a proper `ROM` file for your Card.

The `ROM` file you need is already provided above.

### How To

1. Run the Driver Setup present in the folder.

2. Place the downloaded `ROM` file within the same folder of the `AMDVBFlash` tool - makes things easier.

3. Open a Terminal (or Command Prompt) within that same folder.

4. Run this command: `amdvbflash.exe -i` (This will let you know which slot holds your GPU - <b>DO NOT SKIP</b>).

5. Run this command, using the correct slot: `amdvbflash.exe -unlockrom <slot>`.

6. Flash the provided `ROM` with the following command: `amdvbflash.exe -f -p <slot> <ROM_FILE> --show-progress`.

<br>
After these steps, your GPU <i>should</i> be flashed.

Check the Terminal for errors, and then <b>reboot</b>.

### Finishing Up

Using [GPU-Z](https://www.techpowerup.com/download/techpowerup-gpu-z/), check the data output from your GPU.

It should look like this:

![alt text](https://github.com/danfq/Ryzentosh/blob/main/GPU%20Preparations/success.png?raw=true)


You're done!
<br>
Your `RX580 2048SP` is now MacOS-Ready!

## DRM - Digital Rights Management

### The Issue

From `Big Sur` onwards, DRM is fixed by default.<br>
However, if you're on an older version of macOS, you need to add a "Mode" to your `WhateverGreen` Kext:

- Find the correct value for your System [here](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.Chart.md).
- Add your Mode to your `boot-args`; for example: `shikigva=16` or `shikigva=80` (these usually work despite what the Chart says).

DRM-enabled Services, such as `Netflix` or `Apple TV+`, should work well now.

## PAT (Page Attribute Table)

`PAT` Patches are **already included** with OpenCore.<br>
However, you may choose between `Shaneee` or `Algrey` Patches.

The table below defines both:

| Component   | Shaneee       | Algrey       |
| :---        |    :----:   |          ---: |
| GPU         | Best Performance (May Not Work With All NVIDIA GPUs) | Worse Performance (Works With All GPUs) |
| GPU Audio   | May Not Work   | Works    |
| Enabled         | By Default       | Disabled By Default    |

To change which Patch you'd like to use, search for `fix PAT` in your config.plist.<br>
Set `Enabled` to `true` on your desired Patch.

**DO NOT** enable both Patches at once.<br>
You'll either get a Kernel Panic, or not boot at all.

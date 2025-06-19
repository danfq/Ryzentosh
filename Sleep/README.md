# Sleep

Up until macOS Monterey (Version `12.3+`), Apple used to enforce a USB Port Limit of **15 Ports**.<br>
This is why `OpenCore` provides the `XhciPortLimit` property - which should be set to `true` if you're on an older System (pre Monterey 12.3), and `false` otherwise.

## Preparations

Firstly, check if Sleep works OOB (Out-Of-The-Box) on your System.<br>
If it doesn't, proceed to the fixes below.

### Commands
Run the following commands, to disable the following:

- `sudo pmset autopoweroff 0` - AutoPowerOff (a form of Hibernation).
- `sudo pmset powernap 0` - Periodically wakes up the machine for Network, and to check for Updates (won't turn on the Display).
- `sudo pmset standby 0` - Time Period between Sleep and Hibernation.
- `sudo pmset proximitywake 0` - Wake from iPhone or Apple Watch, when near the machine (optional).
- `sudo pmset tcpkeepalive 0` - Wake every 2 Hours.

### config.plist

1. Set `Misc -> Boot -> HibernateMode` to `None` - This is a String.

2. Add the following to your `boot-args`, at `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82`:

- `keepsyms=1` - Makes sure we get all possible information from possible Kernel Panics during Sleep.
- `swd_panic=1` - Avoids a Reboot on attempting to Sleep, giving us a Kernel Panic Log instead.

### BIOS

Disable:

- `Wake-on-LAN`.
- `TPM` - Trusted Platform Module.
- `Wake-on-USB` - Make sure your Motherboard doesn't require this to Wake (usually, most people will get random Wakes with this enabled).

Enable:

- `Wake-on-Bluetooth` - Only if you're using a Bluetooth Device to wake (such as a Keyboard, iPhone or Apple Watch).

**ATTENTION: Windows Dual-Boot**

If you're dual-booting Windows, using [BitLocker](https://support.microsoft.com/en-us/windows/bitlocker-drive-encryption-76b92ac9-1040-48d6-9f5f-d14b3c5fa178) and disable `TPM`, your Encryption Keys will be **LOST**.

Disable `TPM` at your own risk.

## USB Port Mapping & USB Power

### USB Mapping

The main reason Sleep doesn't work on AMD Systems, is `USB Port Mapping`.

You can map your USB Ports, from `macOS` or `Windows`, using [this](https://github.com/usbtoolbox/tool) tool.

Because macOS limits the amount of information the User can "see", mapping USB Ports from macOS is **NOT RECOMMENDED**.<br>
If you *must* use macOS, I suggest [USBMap](https://github.com/CorpNewt/USBMap) instead.

After re-mapping your USBs, make sure to disable `XhciPortLimit`, add `UTBMap.kext` and `USBToolBox.kext` to your `Kexts` folder, and refresh your config.plist via [ProperTree](https://github.com/corpnewt/ProperTree) or your preferred PList Editor.

### USB Power

With `Skylake` (and newer) SMBios, Apple no longer provides USB Power Settings via `IOUSBHostFamily`.<br>
Because of this, we must adopt the same method real Mac Devices do, and supply macOS with a `USBX` Device - via `SSDT`.

This will set both the Wake and Sleep Power Values for all your USB ports, and can help fix many High Power Devices, such as:

- `Microphones`.
- `DACs` - Digital-to-Analog Converters.
- `Webcams`.
- `Bluetooth Dongles`.

The following SMBios' need `USBX`:

- `iMac17,x` and Newer
- `MacPro7,1` and Newer
- `iMacPro1,1` and Newer
- `Macmini8,1` and Newer
- `MacBook9,x` and Newer
- `MacBookAir8,x` and Newer
- `MacBookPro13,x` and Newer

I recommend you use [SSDTTime](https://github.com/corpnewt/SSDTTime) to generate the SSDT.<br>
If you can't - or don't want to - use this method, you can download a pre-compiled SSDT [here](https://github.com/dortania/OpenCore-Post-Install/raw/refs/heads/master/extra-files/SSDT-USBX.aml).

### GPRW/UPRW/LANC Instant Wake

macOS will Instant Wake if either USB or Power State changes, while Sleeping.<br>
To fix this, we need to re-route `GPRW/UPRW/LANC` calls to an SSDT.

Before applying the SSDT, verify you have these issues by running:<br>
`pmset -g log | grep -e "Sleep.*due to" -e "Wake.*due to"`

The output of the above command will give you some insight into what's causing Instant Wake, such as:

- `Wake [CDNVA] due to GLAN: Using AC` - Generally caused by `Wake-on-Lan`; try disabling that first.
- `Wake [CDNVA] due to HDEF: Using AC` - `HDEF` (your On-Board Audio) prevented Sleep.
- `Wake [CDNVA] due to XHC: Using AC` - Generally caused by `Wake-on-USB` (you likely need a `GPRW` Patch).
- `Wake reason: RTC (Alarm)` - Generally caused by an App preventing Sleep (quitting the App before trying to Sleep should suffice).

| SSDT | When to Use |
| :--- | ---: |
| [GPRW](https://github.com/dortania/OpenCore-Post-Install/raw/refs/heads/master/extra-files/SSDT-GPRW.aml) | Use this if you have `Method GPRW, 2` in your ACPI |
| [UPRW](https://github.com/dortania/OpenCore-Post-Install/raw/refs/heads/master/extra-files/SSDT-UPRW.aml) | Use this if you have `Method UPRW, 2` in your ACPI |
| [LANC](https://github.com/dortania/OpenCore-Post-Install/raw/refs/heads/master/extra-files/SSDT-LANC.aml) | Use this if you have `Device (LANC)` in your ACPI |

## Extra GPU

If you're using the same System as me, or a similar one, chances are that you have an unsupported - and very stubborn - GPU, hanging around.<br>
macOS will still "see" it, and attempt to load Drivers for it.

Since the GPU isn't supported, this means you'll always have an Assertion:<br>
`Idle sleep preventers: IODisplayWrangler`

This Assertion will **always** prevent proper Sleep.

### The Fix

My current fix may be messy, and I may change it in the future - if I find a better way to do this.

But, in short, I needed a few patches.
One `SSDT`, a few `DeviceProperties` entries, and an `ACPI` Patch.

The `SSDT` is available in this folder, and the other patches are below:

#### SSDT-DGPU

This SSDT is a joint effort by me, [T3 Chat](https://t3.chat), and the legendary [Edhawk](https://forum.amd-osx.com/members/edhawk.105/).
It references the PCI bridge (`BRG0`) and disables **all levels** of the PCI path by overriding their `_STA` methods to return `Zero`:

- `GPP8` - The PCI bus.
- `X161` - The PCI slot.
- `BRG0` - The PCI bridge.
- `GFX1` -The GPU itself.


This effectively tells macOS that the **entire device chain** is not present, making the GPU *invisible* to the OS.
It is noteworthy that this SSDT **does not** power down the GPU - it simply *prevents* macOS from loading Drivers for it.

#### DeviceProperties

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DeviceProperties</key>
    <dict>
        <key>Add</key>
        <dict>
            <key>PciRoot(0x0)/Pci(0x3,0x1)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)</key>
            <dict>
                <key>device-id</key>
                <data>AAAAAA==</data>
                <key>vendor-id</key>
                <data>AAAAAA==</data>
                <key>subsystem-id</key>
                <data>AAAAAA==</data>
                <key>subsystem-vendor-id</key>
                <data>AAAAAA==</data>
                <key>class-code</key>
                <data>AAA=</data>
                <key>compatible</key>
                <string>pci0000,0000</string>
                <key>model</key>
                <string>NotAGPU</string>
                <key>name</key>
                <string>NotAGPU</string>
                <key>IOName</key>
                <string>NotAGPU</string>
                <key>IOPCIMatch</key>
                <data>AAAAAA==</data>
                <key>IONameMatch</key>
                <string>NotAGPU</string>
                <key>IOProviderClass</key>
                <string>IOPCIDevice</string>
                <key>hda-gfx</key>
                <string></string>
                <key>AAPL,slot-name</key>
                <string>NotAGPU</string>
                <key>built-in</key>
                <data>AA==</data>
            </dict>
            <key>PciRoot(0x0)/Pci(0x3,0x1)/Pci(0x0,0x0)/Pci(0x0,0x0)</key>
            <dict>
                <key>device-id</key>
                <data>AAAAAA==</data>
                <key>vendor-id</key>
                <data>AAAAAA==</data>
                <key>subsystem-id</key>
                <data>AAAAAA==</data>
                <key>subsystem-vendor-id</key>
                <data>AAAAAA==</data>
                <key>class-code</key>
                <data>AAA=</data>
                <key>compatible</key>
                <string>pci0000,0000</string>
                <key>model</key>
                <string>NotAGPU</string>
                <key>name</key>
                <string>NotAGPU</string>
                <key>IOName</key>
                <string>NotAGPU</string>
                <key>IOPCIMatch</key>
                <data>AAAAAA==</data>
                <key>IONameMatch</key>
                <string>NotAGPU</string>
                <key>IOProviderClass</key>
                <string>IOPCIDevice</string>
                <key>hda-gfx</key>
                <string></string>
                <key>AAPL,slot-name</key>
                <string>NotAGPU</string>
                <key>built-in</key>
                <data>AA==</data>
            </dict>
            <key>PciRoot(0x0)/Pci(0x3,0x1)/Pci(0x0,0x0)</key>
            <dict>
                <key>device-id</key>
                <data>AAAAAA==</data>
                <key>vendor-id</key>
                <data>AAAAAA==</data>
                <key>subsystem-id</key>
                <data>AAAAAA==</data>
                <key>subsystem-vendor-id</key>
                <data>AAAAAA==</data>
                <key>class-code</key>
                <data>AAA=</data>
                <key>compatible</key>
                <string>pci0000,0000</string>
                <key>model</key>
                <string>NotAGPU</string>
                <key>name</key>
                <string>NotAGPU</string>
                <key>IOName</key>
                <string>NotAGPU</string>
                <key>IOPCIMatch</key>
                <data>AAAAAA==</data>
                <key>IONameMatch</key>
                <string>NotAGPU</string>
                <key>IOProviderClass</key>
                <string>IOPCIDevice</string>
                <key>hda-gfx</key>
                <string></string>
                <key>AAPL,slot-name</key>
                <string>NotAGPU</string>
                <key>built-in</key>
                <data>AA==</data>
            </dict>
            <key>PciRoot(0x0)/Pci(0x3,0x1)</key>
            <dict>
                <key>device-id</key>
                <data>AAAAAA==</data>
                <key>vendor-id</key>
                <data>AAAAAA==</data>
                <key>subsystem-id</key>
                <data>AAAAAA==</data>
                <key>subsystem-vendor-id</key>
                <data>AAAAAA==</data>
                <key>class-code</key>
                <data>AAA=</data>
                <key>compatible</key>
                <string>pci0000,0000</string>
                <key>model</key>
                <string>NotAGPU</string>
                <key>name</key>
                <string>NotAGPU</string>
                <key>IOName</key>
                <string>NotAGPU</string>
                <key>IOPCIMatch</key>
                <data>AAAAAA==</data>
                <key>IONameMatch</key>
                <string>NotAGPU</string>
                <key>IOProviderClass</key>
                <string>IOPCIDevice</string>
                <key>hda-gfx</key>
                <string></string>
                <key>AAPL,slot-name</key>
                <string>NotAGPU</string>
                <key>built-in</key>
                <data>AA==</data>
            </dict>
            <key>PciRoot(0x0)/Pci(0x8,0x1)/Pci(0x0,0x4)</key>
            <dict>
                <key>name</key>
                <string>HDEF</string>
                <key>compatible</key>
                <string>pci1022,1487</string>
                <key>device-id</key>
                <data>hxQAAA==</data>
                <key>hda-gfx</key>
                <string>onboard-1</string>
                <key>built-in</key>
                <data>AQ==</data>
                <key>layout-id</key>
                <data>CwAAAA==</data>
                <key>alc-layout-id</key>
                <data>CwAAAA==</data>
            </dict>
        </dict>
        <key>Delete</key>
        <dict/>
    </dict>
</dict>
</plist>
```

The first four entries spoof core values of the GPU, effectively making macOS think there's no GPU at any of the included PCI Paths.

The last entry, for `PciRoot(0x0)/Pci(0x8,0x1)/Pci(0x0,0x4)`, defines core values for the on-board Audio Controller, such as `hda-gfx` and `layout-id`.
For good measure, I also included `alc-layout-id`, but this is supposedly **not needed** (automatically handled by `AppleALC`).

#### ACPI

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<array>
    <dict>
        <key>Comment</key>
        <string>Rename HD Audio AZAL to HDEF</string>
        <key>Enabled</key>
        <true/>
        <key>Find</key>
        <data>QVpBTA==</data>
        <key>Limit</key>
        <integer>0</integer>
        <key>Mask</key>
        <data></data>
        <key>OemTableId</key>
        <data></data>
        <key>Replace</key>
        <data>SERFRg==</data>
        <key>ReplaceMask</key>
        <data></data>
        <key>Skip</key>
        <integer>0</integer>
        <key>TableLength</key>
        <integer>0</integer>
        <key>TableSignature</key>
        <data></data>
    </dict>
</array>
</plist>
```

This patch is pretty self-explanatory:
It renames the on-board `AZAL` device to `HDEF`, which is the name that `AppleALC` expects, to inject the proper Layout ID and Audio Features.
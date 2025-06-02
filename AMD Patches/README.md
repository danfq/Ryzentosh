# AMD Patches

## Permissions

Before running any of these scripts, make sure you give them Execution Permissions, by running:<br>
`sudo chmod +x <script_name>.sh`

### MKL - Math Kernel Library

Many Applications - such as [Discord](https://discord.com) a feature called `MKL - Math Kernel Library`, which is built for `Intel`.

As expected, this feature isn't available on AMD Systems. As a workaround, we can fool macOS into thinking we're running on an Intel CPU.<br>
To do this, run the `fix_mkl.sh` script.

### CPU Name in About This Mac

If you have `Unknown` as your CPU name in `About This Mac`, make sure to set `ProcessorType` (in `PlatformInfo -> Generic`) to the following:

- 8 or More Physical Cores: `3841`.
- Else: `1537`.

### Adobe Products (e.g Photoshop)

Adobe Products also need a fix, on AMD Systems.<br>
This is due to the `intel_fast_memset` instruction which, by definition, isn't available on AMD Systems.

To fix this, run the `fix_adobe.sh` script.<br>
Older Adobe Products (e.g Photoshop until version `22.3.1`) require a different fix: `fix_adobe_legacy.sh`.

Make sure to also run `fix_mkl.sh`.

For more information on Adobe AMD Patching, check out [this](https://macos86.it/topic/4822-wip-photoshop-after-effects-cc-2021-premiere-pro-cc-2021-154-amd-hackintosh-fix/) Thread on `macos86.it`.
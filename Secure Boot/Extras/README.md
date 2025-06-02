# Extras

## Continuity

### The Issue

`macOS`'s `Continuity` feature requires your phone (an iPhone) and your Hackintosh to be on the same Wi-Fi network.

On newer versions of `macOS`, `Broadcom` cards are **no longer supported**, by default.<br>
To make them work, you need to use `OCLP - OpenCore Legacy Patcher`.

The issue, however, is that `OCLP` requires `SecureBootModel` to be set to `Disabled`.

Since that's not possible, running a secure version of `macOS`, you can't go down that route.

### The Fix

To make `Continuity` work as expected, you'll need to run the provided `Continuity Activation Tool`.

After doing so, `Continuity` should work as expected!

Another fix is to apply the `OCLP` Patches **before** enabling `Secure Boot`, `SIP` and `AMFI`.
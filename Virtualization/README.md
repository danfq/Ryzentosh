# Virtualization

On AMD Systems, `Virtualization` is quite broken.<br>
Check this Compatibility Table for more information:

| Software | Compatibility |
| :--- | ---: |
| VirtualBox | Drastically Decreased Performance |
| VMWare Fusion 10 | Only Catalina and Older - For Catalina Use [this](https://posts.boy.sh/vmware-fusion-catalina) Patch |
| Docker | Only `Docker in VirtualBox` or `Docker Toolbox` |
| Parallels Desktop | Up to 13.1 Unless AppleHV is Used |
| Android Emulator | Only `Android-x86` with Compatible VM Software |
| iOS Emulator | Works OOB (Out-Of-the-Box) |

## Resource Management - Parallels

Regardless of your Hardware, you'll have Performance Issues with Virtual Machines.<br>
As such, the following are the best Hardware Configurations:

- CPU: `4 Cores`.
- RAM: `4GB` to `8GB`.
- VRAM: `1GB`.
- 3D Acceleration: `DirectX 9`.
- OS: `Windows 7` (`SP1 Build 7601`) with `Aero` Disabled.
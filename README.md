# Notes
Big Sur 11.2.3

## Hardware
```
Asus-Maximus-IX-CODE
i7 6700K (Intel Framebuffer - reference)
GTX 960
TL-WDN4800
```

## BIOS config
### High Sierra with Nvidia GTX 960 / Big Sur with MSI RX 570 Armor
```
Exit
- Load Optimized Defaults
Boot Menu
- Fast Boot -> Disabled
- Secure Boot -> OS Type -> Other OS
Advanced
- System Agent (SA) Configuration -> VT-d -> Disable
- System Agent (SA) Configuration -> Graphics Configuration -> Primary Display -> Auto
- System Agent (SA) Configuration -> Graphics Configuration -> iGPU Multi-Monitor -> Enable
- PCH Configuration -> IOAPIC 24-119 -> Disabled
- Onboard Devices Configuration -> Wi-Fi Controller -> Disabled (Bluetooth Controller is Enabled -  naitive support but Wifi is not - replaced by TL-WDN4800)
Extreme Tweaker
- Ai Overclocker Tuner -> XMP
- Extreme Tweeking -> Enable
```
### Catalina with Intel HD 530 (cannot use dual monitors)
```
Exit
- Load Optimized Defaults
Boot Menu
- Fast Boot -> Disabled
- Secure Boot -> OS Type -> Other OS
Advanced
- System Agent (SA) Configuration -> VT-d -> Disable
- System Agent (SA) Configuration -> Graphics Configuration -> Primary Display -> iGPU
- PCH Configuration -> IOAPIC 24-119 -> Disabled
- Onboard Devices Configuration -> Wi-Fi Controller -> Disabled (Bluetooth Controller is Enabled - naitive support but Wifi is not - replaced by TL-WDN4800)
Extreme Tweaker
- Ai Overclocker Tuner -> XMP
- Extreme Tweeking -> Enable
```

## Boot logs
```
log show --predicate "processID == 0" --debug
log show --predicate "processID == 0" --start $(date "+%Y-%m-%d") --debug
bdmesg
bdmesg | more
# Debug logs of WhateverGreen/Lilu
log show --predicate 'process == "kernel" AND (eventMessage CONTAINS "WhateverGreen" OR eventMessage CONTAINS "Lilu")' --style syslog --source --last boot >weglog.txt
# To check if SSDTs are loading:
bdmesg | grep -y aml
```

## Disable Gate Keeper and mount the System Partition as Read/Write
```
sudo spctl --master-disable
sudo mount -uw
sudo killall Finder
```

## Rebuild kexts
```
sudo chown -v -R root:wheel /System/Library/Extensions
sudo touch /System/Library/Extensions
sudo chmod -v -R 755 /Library/Extensions
sudo chown -v -R root:wheel /Library/Extensions
sudo touch /Library/Extensions
sudo kextcache -i /
```

# Kexts

## /EFI/CLOVER/kexts/Other
```
AppleALC.kext
IntelMausiEthernet.kext
Lilu.kext
USBPorts.kext
VirtualSMC.kext
WhateverGreen.kext
```

## Compile kexts

Prepare xcode
```
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

```
# Lilu
cd ~/workspace/HACKINTOSH/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/Lilu.git
git clone https://github.com/acidanthera/MacKernelSDK
cd Lilu
cp -R ./../MacKernelSDK .
xcodebuild -configuration Debug
cp -R build/DEBUG/Lilu.kext ./../
xcodebuild
cp -R build/Release/Lilu.kext ./../kexts/

# WhateverGreen
cd ~/workspace/HACKINTOSH/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/WhateverGreen.git && cd WhateverGreen
cp -R ./../MacKernelSDK ./../Lilu.kext .
xcodebuild
cp -R build/Release/WhateverGreen.kext ./../kexts/

# VirtualSMC
cd ~/workspace/HACKINTOSH/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/VirtualSMC.git && cd VirtualSMC
cp -R ./../MacKernelSDK ./../Lilu.kext .
xcodebuild
cp -R build/Release/VirtualSMC.kext build/Release/SMCProcessor.kext build/Release/SMCSuperIO.kext ./../kexts/

# AppleALC
cd ~/workspace/HACKINTOSH/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/AppleALC.git && cd AppleALC
cp -R ./../MacKernelSDK ./../Lilu.kext .
xcodebuild
cp -R build/Release/AppleALC.kext ./../kexts/

# BrcmPatchRAM3
cd ~/workspace/HACKINTOSH/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/BrcmPatchRAM.git && cd BrcmPatchRAM
cp -R ./../MacKernelSDK ./../Lilu.kext .
xcodebuild
cp -R build/Products/Release/BrcmBluetoothInjector.kext build/Products/Release/BrcmFirmwareData.kext build/Products/Release/BrcmPatchRAM3.kext ./../kexts/

# AirportBrcmFixup
cd ~/workspace/HACKINTOSH/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/AirportBrcmFixup.git && cd AirportBrcmFixup
cp -R ./../MacKernelSDK ./../Lilu.kext .
xcodebuild
cp -R build/Release/AirportBrcmFixup.kext ./../kexts/

# IntelMausi
cd ~/workspace/HACKINTOSH/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/IntelMausi.git && cd IntelMausi
cp -R ./../MacKernelSDK .
xcodebuild
cp -R build/Release/IntelMausi.kext ./../kexts/
```

#  Clover EFI bootloader
* Latest version - 10.15.4 (19E266) requires Clover bootloader 5017 or above

# Install
* Create USB using createinstallmedia method
* Install Clover Bootloader to USB
* Copy VirtualSmc.efi, HFSPlus.efi (High Sierra) from /drivers to /Clover/drivers/UEFI/

# Post Install
* Install Clover Bootloader
* Upgrade latest security patch
* Install Nvidia Web Driver 17G11023 - 387.10.10.10.40.134 <- High Sierra with Nvidia GTX 960
* Generate SMBIOS, CustomUUID
* Remove nv_disable=1 and other debug agurments from boot menu <- High Sierra with Nvidia GTX 960
* Enable NvidiaWeb flag <- High Sierra with Nvidia GTX 960
* Copy VirtualSmc.efi, HFSPlus.efi (High Sierra) from /drivers to /Clover/drivers/UEFI/

* Run Apps from internet (Catalina):
```
sudo spctl --master-disable
```
* Test POWER MANAGEMENT:
```
# Drag AppleIntelInfo.kext to desktop
sudo -s
chown -R 0:0 ~/Desktop/AppleIntelInfo.kext
chmod -R 755 ~/Desktop/AppleIntelInfo.kext
kextload ~/Desktop/AppleIntelInfo.kext
cat /tmp/AppleIntelInfo.dat
kextunload ~/Desktop/AppleIntelInfo.kext
```
* Disable Hibernation
```
sudo pmset -a hibernatemode 0
sudo rm /var/vm/sleepimage
sudo mkdir /var/vm/sleepimage
```
* Disable the other hibernation/sleep related options:
```
sudo pmset autopoweroff 0
sudo pmset powernap 0
sudo pmset standby 0
sudo pmset proximitywake 0
sudo pmset tcpkeepalive 0
```

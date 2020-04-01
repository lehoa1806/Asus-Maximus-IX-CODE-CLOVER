# Notes
Calatina 10.15.4 (19E266)

## Hardware
```
Asus-Maximus-IX-CODE  # This config works well with Giga Z170 D3H
i7 6700K (Intel Framebuffer - reference)
GTX 960
TL-WDN4800
```

## BIOS config
### High Sierra with Nvidia GTX 960 / Catalina with MSI RX 570 Armor
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
IO80211Family.kext # Catalina
IntelMausiEthernet.kext
Lilu.kext
USBPorts.kext
VirtualSMC.kext
WhateverGreen.kext
```
IO80211Family.kext <- Original kext from High Sierra 13.6, kext for TL-WDN4800 N900, which is removed from Catalina

## Compile kexts

Prepare xcode
```
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

```
# Unzip kexts in /kexts
mkdir -p /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build/kexts
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build/kexts
mkdir DEBUG
mkdir RELEASE

# Build DEBUG ========================================================
# Lilu
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/Lilu.git && cd Lilu
xcodebuild -configuration Debug
cp -R build/Debug/Lilu.kext ./../kexts/DEBUG/

# AppleALC
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/AppleALC.git && cd AppleALC
cp -R ../Lilu/build/Debug/Lilu.kext .
xcodebuild -configuration Debug
cp -R build/Debug/AppleALC.kext ./../kexts/DEBUG/

# VirtualSMC
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/VirtualSMC.git && cd VirtualSMC
cp -R ../Lilu/build/Debug/Lilu.kext .
xcodebuild -configuration Debug
cp -R build/Debug/VirtualSMC.kext ./../kexts/DEBUG/

# WhateverGreen
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build
git clone https://github.com/acidanthera/WhateverGreen.git && cd WhateverGreen
cp -R ../Lilu/build/Debug/Lilu.kext .
xcodebuild -configuration Debug
cp -R build/Debug/WhateverGreen.kext ./../kexts/DEBUG/

# IntelMausiEthernet
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build
git clone https://github.com/Mieze/IntelMausiEthernet.git && cd IntelMausiEthernet
xcodebuild -configuration Debug
cp -R build/Debug/IntelMausiEthernet.kext ./../kexts/DEBUG/

# IO80211Family and USBInjectAll
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build
cp -R ./../kexts/IO80211Family.kext kexts/DEBUG/
cp -R ./../kexts/USBPorts.kext kexts/DEBUG/


# Build RELEASE ========================================================
# Lilu
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build/Lilu
xcodebuild
cp -R build/Release/Lilu.kext ./../kexts/RELEASE/

# AppleALC
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build/AppleALC
xcodebuild
cp -R build/Release/AppleALC.kext ./../kexts/RELEASE/

# VirtualSMC
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build/VirtualSMC
xcodebuild
cp -R build/Release/VirtualSMC.kext ./../kexts/RELEASE/

# WhateverGreen
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build/WhateverGreen
xcodebuild
cp -R build/Release/WhateverGreen.kext ./../kexts/RELEASE/

# IntelMausiEthernet
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build/IntelMausiEthernet
xcodebuild
cp -R build/Release/IntelMausiEthernet.kext ./../kexts/RELEASE/

# IO80211Family and USBInjectAll
cd /Volumes/HACKINTOSH/workspace/Asus-Maximus-IX-CODE/build
cp -R ./../kexts/IO80211Family.kext kexts/RELEASE/
cp -R ./../kexts/USBPorts.kext kexts/RELEASE/
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
* Disable the other hibernation related options:
```
sudo pmset -a standby 0
sudo pmset -a autopoweroff 0
```
* Wake-up from sleep fix:
```
sudo pmset -a autopoweroff 0
```

### EC0 to EC
>While renaming EC0 to EC might potentially work initially,
>it connects an incompatible driver (AppleACPIEC) to your hardware.
>This can make your system unbootable at any time or hide bugs that
>could trigger randomly.
Use a fake EC device with SSDT-EC
```
https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EC.dsl
```

### _OSI to XOSI
Check IORegistryExplorer when you plug a USB3 to a USB3 port.
If the USB is showed up under PCI0/AppleACPIPCI/XHC/XHC/HS0x, you will need this patch:
```
<dict>
    <key>Comment</key>
    <string>change _OSI to XOSI</string>
    <key>Find</key>
    <data>X09TSQ==</data>
    <key>Replace</key>
    <data>WE9TSQ==</data>
</dict>
```

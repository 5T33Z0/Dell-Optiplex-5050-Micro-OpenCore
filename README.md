# Dell Optiplex 5050 Micro OpenCore
[![Dell](https://img.shields.io/badge/Dell-Optiplex_5050_Micro-informational.svg)](https://dl.dell.com/topicspdf/optiplex-5050-desktop_owners-manual2_en-us.pdf) [![OpenCore](https://img.shields.io/badge/OpenCore-1.0.5-cyan.svg)](https://github.com/acidanthera/OpenCorePkg/releases/latest) ![MacOS](https://img.shields.io/badge/macOS-14.7.5–15.5-purple.svg) [![release](https://img.shields.io/badge/Download-latest-success.svg)](https://github.com/5T33Z0/Dell-Optiplex-5050-Micro-OpenCore/releases)<br>

## About

OpenCore EFI Folder for the **Dell OptiPlex 5050 Micro Small Form Factor** Desktop with an Intel® Core™ i5-7500T. This model has no Wi-Fi/BT module (yet) – it's connected via LAN. I am basically using it as a streaming box connected to my TV via HDMI controlled via a Wireless Keyboard (Logitec 400k+).

I've generated the base EFI folder with OpCore Simplify but I had to modify and tweak it a bit: 

- I added IRQ fixes for working audio 
- Added a proper framebufer patch so that on-bord graphics acceleration would work. 
- Added a modified DMAR Tabel with stripped Reserved Memory Regions so that the `DisableIOMaper` quirk is not required and `AppleVTD` does work.

It's currently using the `iMac18,3` SMBIOS because that's the closest in terms of CPU specs. The `macmini8,3` SMBIOS uses an 8th Gen CPU, so it's not really suited for this system.

## System Specs

**Compenent**   | Description 
----------------|:--------------------------------|
**Model**       | Dell OptiPlex 5050 Micro
**Regulatory Type** | D10U002
**CPU**         | 7th Gen Intel® Core™ i5-7500T (Kaby Lake) 
**GPU**         | Intel HD Graphics 630 
**RAM**         | 8GB 2400MHz DDR4 
**Storage**     | 120 GB SATA SSD      
**NIC**         | Intel I219-LM
**WiFi/BT**     | N.A. 
**SMBIOS**      | `iMac18,3`       

## What works?

### macOS (tested)

- [x] macOS Sequoia
- [x] macOS Sonoma

### Hardware

- [x] iGPU (Intel HD Graphics 630)
- [x] SATA drive
- [x] USB 3.1 (XHCI)
- [x] Ethernet
- [x] Audio
  
## Issues

- [ ] Black Screen on wake: This has to be addressed. Adding the `force-online` property to the framebuffer patch didn't help. The workaround for now is to prohibit the display form turingn off when the system is idle (see System Settings &rarr; Lock Screen) 

## BIOS Settings

- **General**
	- **Boot Sequence**: Add a path to the `OpenCore.efi` (Add Boot Option)
	- **UEFI Boot Path Security**: Never
- **System Configuration**
	- **Integrated NIC**
		- Enable Network Stack [x]
		- Enabled
	- **SATA Operation**: AHCI
	- **Drives**: Enable the Interfaces you want/need (SATA/M.2)
	- **USB Configuration**: enable the USB ports you want/need (no USB kext required, since it's within the 15 port Limit, defined as the correct types already)
- **Video**
	- Primayry Display: Intel HD Graphics
- **Secure Boot**
	- Seucre Boot Enable: Disabled
- **Intel Software Guard Extension**
	- Intel SGX Enable: Disabled
- **Performance**
	- Multicore Support: All
	- Intel Speedstepp: Enable
	- C-States: Enable
	- Intel TurbBoost: enable
- **Power Management**
	- AC Recovery: Power Off
	- Disk Sleep Control: Enable S4 and S5
	- USB Wake Support: Enable
	- Wake on LAN/WLAN: Disabled
- **POST Behavior**
	- Fastboot: Thorough	 
- **Virtualization Support**
	- Enable Intel Virtualization Technology: enable if you want to use Hyper-V (Windows)
	- VT for Direct I/O: enable if you want to use Hyper-V (Windows)
- **Wireless**:
	-  WLAN/Wiig: if you have a WLAN card, enable it. 	-  Bluetooth: if you have a BT Card, enable it
- **Advanced Configurations**
	- ASPM: Auto

> [!NOTE]
> 
> If your Dell Optiplex has a WiFI/BT card you still have to add the required kexts and settings (and maybe OCLP root patches) to make it work! 

## Deployment
- Download the latest OC EFI folder from the "Releases" section
- Extract it
- Open the config.plist with OCAT
- Select the PlatformInfo/Generic Section and Generate a Serial number
- Copy EFI to a FAT32 formtted USB flash drive
- Boot macOS from the USB flash drive
- If the folder works then copy it to your internal disk

## Credits
- lzhoang2801 for [**OpCore Simplify**](https://github.com/lzhoang2801/OpCore-Simplify)
- Coopydood for the Framebuffer patch in his [**Dell 7050 Micro OC Repo**](https://github.com/Coopydood/OpenCore-OptiPlex7050-Micro/)
- CorpNewt for [**SSDTTime**](https://github.com/corpnewt/SSDTTime)

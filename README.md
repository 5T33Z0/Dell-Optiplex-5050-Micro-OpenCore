# Dell Optiplex 5050 Micro OpenCore
[![Dell](https://img.shields.io/badge/Dell-Optiplex_5050_Micro-informational.svg)](https://dl.dell.com/topicspdf/optiplex-5050-desktop_owners-manual2_en-us.pdf) [![OpenCore](https://img.shields.io/badge/OpenCore-1.0.7-cyan.svg)](https://github.com/acidanthera/OpenCorePkg/releases/latest) ![MacOS](https://img.shields.io/badge/macOS-14.7.5–26.2-purple.svg) [![release](https://img.shields.io/badge/Download-latest-success.svg)](https://github.com/5T33Z0/Dell-Optiplex-5050-Micro-OpenCore/releases)

![optiplex5050](https://github.com/user-attachments/assets/b9b0908a-2a02-46fc-9964-239fdfd2c4a8)

----

**TABLE of CONTENTS**

- [About](#about)
- [System Specs](#system-specs)
- [What works?](#what-works)
	- [macOS (tested)](#macos-tested)
	- [Hardware](#hardware)
- [Issues](#issues)
- [BIOS Settings](#bios-settings)
- [Deployment](#deployment)
- [Post-Install](#post-install)
	- [Fixing Sleep issues](#fixing-sleep-issues)
	- [Disable Gatekeeper (optional)](#disable-gatekeeper-optional)
- [Credits](#credits)

---

## About

OpenCore EFI Folder for the **Dell OptiPlex 5050 Micro Small Form Factor** Desktop with an Intel® Core™ i5-7500T. This model has no Wi-Fi/BT module (yet) – it's connected via LAN. I am basically using it as a streaming box connected to my TV via HDMI controlled via a Wireless Keyboard (Logitec 400k+). I get 4k resolution at 60 Hz in Windows – I can't seem to enable it in macOS, though (no LSPCON).

I've generated the base EFI folder with OpCore Simplify but I had to modify and tweak it a bit: 

- Added IRQ fixes so audio works 
- Added a proper framebufer patch so that on-bord graphics acceleration would work. 
- Added a modified DMAR table with stripped Reserved Memory Regions so that the `DisableIOMaper` quirk is not required and `AppleVTD` does work.

It's currently using the `iMac18,3` SMBIOS because that's the closest in terms of CPU specs. The `macmini8,3` SMBIOS uses an 8th Gen CPU, so it's not really suited for this system.

|⚠️ Important Updates|
|:--------------------------|
| Works with macOS Tahoe beta

## System Specs

**Compenent**   | Description 
---------------:|:--------------------------------|
**Model**       | Dell OptiPlex 5050 Micro
**Regulatory Type** | D10U002
**Chipset**     | Intel [Q270](https://www.intel.com/content/www/us/en/products/sku/98088/intel-q270-chipset/specifications.html)
**CPU**         | 7th Gen Intel® Core™ i5-7500T (Kaby Lake) 
**GPU**         | Intel HD Graphics 630
**Sound**       | Realtek ALC255 <br> Layout: `11`
**RAM**         | 8GB 2400MHz DDR4 
**Storage**     | Samsung SSD 860 EVO 250GB     
**NIC**         | Intel I219-LM
**USB**         | 6 USB Type A Ports <br> Spec: USB 3.1 Gen 1 (up to 5 Gbps)
**WiFi/BT**     | N.A. 
**SMBIOS**      | `iMac18,3`       

**More Info**: https://www.hardware-corner.net/desktop-models/Dell-OptiPlex-5050M/<br>
**Manual**: https://dl.dell.com/topicspdf/optiplex-5050-desktop_owners-manual2_en-us.pdf

>[!NOTE]
>
> **WiFi**: I've tested Wi-Fi with a TP Link [Archer T2U Nano](https://www.tp-link.com/de/home-networking/adapter/archer-t2u-nano/) USB dongle and this [tool](https://github.com/chris1111/Wireless-USB-OC-Big-Sur-Adapter) in macOS Sonoma and Sequoia and it's working fine.

## What works?

### macOS (tested)

- [x] macOS Tahoe beta
- [x] macOS Sequoia
- [x] macOS Sonoma

### Hardware

- [x] iGPU (Intel HD Graphics 630)
	- [x] HDMI 1.4 Port (`con0`)
	- [x] Display Port (`con1`)
	- [x] Display Port (`con2`)  
- [x] SATA Controller
- [x] USB 3.1 (XHCI). Only 12 ports in total are enabled in the OEM USB SSDT, so no port mapping is required!
- [x] Ethernet
- [x] Audio (Line-Out, Mic, Headphone, Built-in Speaker)

> [!NOTE]
>
> Display Port `con2` is an optional expansion not present on stock devices (Dell Micro_DP Upgrade A00 LX4038).
  
## Issues

- [ ] Black Screen on wake – displays won't turn on after exiting sleep. Adding the `force-online` property to the framebuffer patch didn't help. The workaround for now is to prohibit the display form turning off when the system is idle (see System Settings &rarr; Lock Screen)
- [ ] No 4k resolution in macOS (I either can't find the required LPSCON to enable 4K output or doesn't have one)

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
	- Primary Display: Intel HD Graphics
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
- Download the latest OC EFI folder from the [Releases](https://github.com/5T33Z0/Dell-Optiplex-5050-Micro-OpenCore/releases) section
- Extract it
- Download [**OCAT**](https://github.com/ic005k/OCAuxiliaryTools) and run it
- In the menu bar select "Edit" > "OpenCore DEV" to change the OpenCore version
- Ignore the warning about missing files
- Next, click on "Upgrade OpenCore and Kexts"
- In the "Sync" Window, click on "Get OpenCore" to download the latest OC build
- Close the sync window
- Back in the Main window click on the button to mount the EFI partiton
- Click on "Mount and open config.plist"
- Select the PlatformInfo/Generic Section and click on "Generate" next to the "SystemProductName" dropdown menu
- Copy EFI to a FAT32 formtted USB flash drive
- Boot macOS from the USB flash drive
- If the folder works then copy it to your internal disk

## Post-Install

### Fixing Sleep issues

In order to prevent the most common issues with sleep, we will set it to `hibernatemode 0` (Suspend to RAM), write protect the slee pimage using Terminal:


```bash
sudo pmset -a hibernatemode 0
sudo rm /var/vm/sleepimage
sudo touch /var/vm/sleepimage
sudo chflags uchg /var/vm/sleepimage
```

Next, we disable `displaysleep` and `powernap` to workaround the black-screen-on-wake issue. And since this Mini-PC does not have a motion sensor, we also disable proximitywake: 

```bash 
sudo pmset displaysleep 0
sudo pmset powernap 0
sudo pmset proximitywake 0
```

### Disable Gatekeeper (optional)
I disable Gatekeeper on my systems because it is annoying and wants to stop you from running scripts from github etc. To do so, enter `sudo spctl --master-disable` in Terminal. Disabling Gatekeeper in macOS Sequoia requires a few more [steps](https://github.com/5T33Z0/OC-Little-Translated/blob/main/14_OCLP_Wintel/Guides/Disable_Gatekeeper.md).

### Enable Apple TV and Apple Music streaming
- Add `unfairgva=4` to boot-args and restart the system. Afterwards Apple TV and Music will work (if you have a subscription). 

## Credits
- lzhoang2801 for [**OpCore Simplify**](https://github.com/lzhoang2801/OpCore-Simplify)
- Coopydood for the Framebuffer patch in his [**Dell 7050 Micro OC Repo**](https://github.com/Coopydood/OpenCore-OptiPlex7050-Micro/)
- CorpNewt for [**SSDTTime**](https://github.com/corpnewt/SSDTTime)

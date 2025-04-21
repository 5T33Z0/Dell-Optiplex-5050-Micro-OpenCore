# Dell Optiplex 5050 Micro OpenCore
[![Dell](https://img.shields.io/badge/Dell-Optiplex_5050_Micro-informational.svg)](https://dl.dell.com/topicspdf/optiplex-5050-desktop_owners-manual2_en-us.pdf) [![OpenCore](https://img.shields.io/badge/OpenCore-1.0.5-cyan.svg)](https://github.com/acidanthera/OpenCorePkg/releases/latest) ![MacOS](https://img.shields.io/badge/macOS-14.7.5–15.5-purple.svg) [![release](https://img.shields.io/badge/Download-latest-success.svg)]()<br>

## About

OpenCore EFI Folder for the **Dell OptiPlex 5050 Micro Small Form Factor** Desktop with an Intel® Core™ i5-7500T. This model has no Wi-Fi/BT module (yet). I will add one later but for now it's connected via LAN. I am basically using it as a streaming box connected to my TV via HDMI controlled via a Logitec 400k+ Wireless Keyboard.

I've generated the base EFI folder with OpCore Simplify but I had to modify and tweak it a bit: it needed IRQ fixes for working audio and a proper framebufer patch so that on-bord graphics acceleration would work. It's currently configured as an iMac18,3 because that's the closest in terms of CPU specs. The macmini8,3 uses an 8th Gen CPU, so it's not suited so well for this system, imo.

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
**WiFi/BT**     | None 
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

- [ ] Monitor Wake has to be addressed

## BIOS Settings
…todo…

## Deployment
…todo…

## Credits
- lzhoang2801 for [**OpCore Simplify**](https://github.com/lzhoang2801/OpCore-Simplify)
- Coopydood for the Framebuffer patch in his [**Dell 7050 Micro OC Repo**](https://github.com/Coopydood/OpenCore-OptiPlex7050-Micro/)
- CorpNewt for [**SSDTTime**](https://github.com/corpnewt/SSDTTime)

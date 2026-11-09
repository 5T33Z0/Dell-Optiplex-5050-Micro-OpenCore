# Dell OptiPlex 5050 – Intel HD 630 Framebuffer Patch

## Hardware

- **System:** Dell OptiPlex 5050
- **CPU:** Intel Core i5-7500T
- **iGPU:** Intel HD Graphics 630 (non-UHD)
- **Device ID:** `0x5912`

## Initial Issue

The original framebuffer patch, which I found in this [InsanelyMac thread](https://www.insanelymac.com/forum/topic/362800-success-dell-optiplex-5050-sff-on-macos-tahoe-w-igpu-i7-7700-bcm-wifi-efi-included/), was incorrect:

![](https://www.hackintosh-forum.de/attachment/227968-bildschirmfoto-2026-11-05-um-21-56-37-png/)

The device ID configuration effectively contained a framebuffer patch. Interestingly, it still worked through the DisplayPort output, but HDMI was not functional.

My initial idea was to replace the `6` in the device ID with a `0` to correct the device ID. However, macOS Tahoe crashed during boot with this modification.

After several attempts, I consulted the OpenCore Install Guide and found the following information in the [Kaby Lake Laptop section](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/kaby-lake.html#configuration-notes):

> HD 6xx users (UHD 6xx users are not concerned) may have some issues with output where plugging in a display out would cause a lock up (kernel panic); here are some patches to mitigate that (credit RehabMan).

The guide provides alternative framebuffer configurations to address these issues.

## Base Framebuffer Configuration

Based on the recommendations in the OpenCore Install Guide, I selected the following base framebuffer, which is recommended for NUC configurations:

**`AAPL,ig-platform-id`: `591B0000`**

I chose this personality because `59120000` only defines a single connector, whereas `591B0000` provides a suitable starting point for configuring multiple display outputs.

I initially added the default connector properties:

| Property | Type | Value |
|---|---|---|
| framebuffer-con1-enable | Data | `01000000` |
| framebuffer-con1-alldata | Data | `01050A00 00080000 87010000` |
| framebuffer-con2-enable | Data | `01000000` |
| framebuffer-con2-alldata | Data | `02040A00 00080000 87010000` |

With this configuration, the system booted successfully and HDMI was working. However, DisplayPort was still not functional.

## Identifying the Physical Connectors from the VBT

To investigate the display topology, I extracted the connector information from the system's `vbt.bin` using my application [**vbt2fbpatch**](https://github.com/5T33Z0/vbt2fbpatch).

The VBT analysis revealed five display-device entries, including two DP++ ports:

```text
Input        : /tmp/vbt_1794120480130_yyonea.bin (6144 bytes), VBT header version 100, BDB 209
VBT checksum : 0x0A (OK), child device size 38

Display devices defined in the VBT:

+------+--------+------+-----+------+----------+----------+------+-----+------+-------+
| Slot | Handle | DVO  | DDI | Type | dev_type | AUX      | DDC  | I2C | HPD  | Notes |
+------+--------+------+-----+------+----------+----------+------+-----+------+-------+
| 0    | LFP1   | 0x0A | A   | eDP  | 0x1806   | A (0x40) | 0x03 | -   | st=0 | -     |
| 1    | EFP1   | 0x01 | B   | HDMI | 0x60D2   | -        | 0x05 | -   | st=0 | -     |
| 2    | EFP2   | 0x08 | C   | DP++ | 0x60D6   | C (0x20) | 0x04 | -   | st=0 | -     |
| 3    | EFP3   | 0x09 | D   | DP++ | 0x60D6   | D (0x30) | 0x06 | -   | st=0 | -     |
| 4    | EFP4   | 0x0B | E   | DP   | 0x68C6   | A (0x40) | -    | -   | st=0 | -     |
+------+--------+------+-----+------+----------+----------+------+-----+------+-------+

[i] BDB block 51 runs past the end of the BDB; blocks after it are ignored.
```

The VBT describes the following external display connections:

- **EFP1 (DDI B):** HDMI, DDC `0x05`
- **EFP2 (DDI C):** DP++, AUX C, DDC `0x04`
- **EFP3 (DDI D):** DP++, AUX D, DDC `0x06`
- **EFP4 (DDI E):** DisplayPort, AUX A

For the framebuffer patch, I focused on the HDMI and DisplayPort outputs I intended to use.

### DDC and macOS Bus-ID Correlation

One particularly interesting observation was the relationship between the VBT's DDC assignments and the macOS framebuffer `bus-id` values.

In my testing, the DDC values corresponded directly to the bus IDs that produced working display outputs:

| Physical output | VBT DDC | macOS framebuffer bus-id |
|---|---|---|
| HDMI (EFP1) | `0x05` | `0x05` |
| DisplayPort (EFP2) | `0x04` | `0x04` |

This suggested that the VBT's DDC assignment could be used to derive the framebuffer bus ID. Based on this observation, I retained bus ID `0x05` for HDMI and used bus ID `0x04` for DisplayPort. I also adjusted the connector properties for the second output to reflect the DisplayPort connection.

## Resulting Framebuffer Patch

The resulting configuration successfully enabled both HDMI and DisplayPort.

![FBPatch](https://www.hackintosh-forum.de/attachment/228010-bildschirmfoto-2026-11-09-um-16-32-05-png/)

### DeviceProperties

```xml
<dict>
    <key>PciRoot(0x0)/Pci(0x2,0x0)</key>
    <dict>
        <!-- Intel HD Graphics 630: Kaby Lake framebuffer personality -->
        <key>AAPL,ig-platform-id</key>
        <data>AAAbWQ==</data> <!-- 591B0000 -->

        <!-- Disable Apple Graphics Device Control -->
        <key>disable-agdc</key>
        <data>AQAAAA==</data>

        <!-- Enable HDMI 2.0 support -->
        <key>enable-hdmi20</key>
        <data>AQAAAA==</data>

        <!-- Connector 1: HDMI (EFP1, DDI B, VBT DDC 0x05) -->
        <!-- Bus-ID: 0x05, Pipe: 0x0A, Connector Type: HDMI -->
        <key>framebuffer-con1-alldata</key>
        <data>AQUKAAAIAADHAwAA</data>
        <key>framebuffer-con1-enable</key>
        <data>AQAAAA==</data>

        <!-- Connector 2: DisplayPort (EFP2, DDI C, VBT DDC 0x04) -->
        <!-- Bus-ID: 0x04, Pipe: 0x0A -->
        <key>framebuffer-con2-alldata</key>
        <data>AgQKAAAIAADHAwAA</data>
        <key>framebuffer-con2-enable</key>
        <data>AQAAAA==</data>

        <!-- Enable framebuffer connector patching -->
        <key>framebuffer-patch-enable</key>
        <data>AQAAAA==</data>

        <!-- Set framebuffer stolen memory -->
        <key>framebuffer-stolenmem</key>
        <data>AAAwAQ==</data> <!-- 0x00013000 -->

        <!-- Enable Intel graphics power management control -->
        <key>rps-control</key>
        <data>AQAAAA==</data>
    </dict>
</dict>
```

## Conclusion

This experiment demonstrates that extracting display-device information from the VBT can be useful when constructing Intel framebuffer patches.

In particular, the DDC values provided a direct match for the working macOS framebuffer bus IDs in this configuration:

- HDMI: DDC `0x05` → bus ID `0x05`
- DisplayPort: DDC `0x04` → bus ID `0x04`

This suggests a potential method for automatically deriving framebuffer bus IDs from VBT information.

It would be interesting to investigate whether this relationship also holds across other Intel GPU generations and platforms.

**Further testing is required before treating DDC-to-bus-ID translation as a universal rule.**

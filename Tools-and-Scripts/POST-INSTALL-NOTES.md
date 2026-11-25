# Post-Install Tuning & Reference Snippets

Quick reference commands and configuration snippets for finalizing macOS High Sierra (`10.13.6`) on the Dell Latitude E6400.

---

## 1. AGPM GPU Power Management (`control-id` 18)

By default, the **NVIDIA Quadro NVS 160M** (`Vendor ID: 0x10de`, `Device ID: 0x06eb`) stays locked at `580 MHz` under macOS. Setting `control-id` to `18` enables dynamic stepping across `100 MHz` (idle), `275 MHz` (intermediate), and `580 MHz` (load).

Instead of modifying `/System/Library/Extensions/AppleGraphicsPowerManagement.kext` directly (which gets overwritten by macOS security updates), create a lightweight **codeless injector kext** (e.g., `AGPM-E6400.kext`) in `EFI/OC/Kexts` (or `EFI/CLOVER/kexts/Other`) with an `Info.plist` targeting your active SMBIOS model (such as `MacBookPro5,1` or `MacBookPro6,2`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDevelopmentRegion</key>
    <string>English</string>
    <key>CFBundleIdentifier</key>
    <string>com.custom.AGPM-E6400</string>
    <key>CFBundleInfoDictionaryVersion</key>
    <string>6.0</string>
    <key>CFBundleName</key>
    <string>AGPM-E6400</string>
    <key>CFBundlePackageType</key>
    <string>KEXT</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0.0</string>
    <key>CFBundleVersion</key>
    <string>1.0.0</string>
    <key>IOKitPersonalities</key>
    <dict>
        <key>AGPM</key>
        <dict>
            <key>CFBundleIdentifier</key>
            <string>com.apple.driver.AGPM</string>
            <key>IOClass</key>
            <string>AGPMController</string>
            <key>IONameMatch</key>
            <string>AGPMEnabler</string>
            <key>IOProviderClass</key>
            <string>IOPlatformPluginDevice</string>
            <key>Machines</key>
            <dict>
                <key>MacBookPro5,1</key>
                <dict>
                    <key>Vendor10deDevice06eb</key>
                    <dict>
                        <key>Heuristic</key>
                        <dict>
                            <key>ID</key>
                            <integer>0</integer>
                            <key>IdleInterval</key>
                            <integer>250</integer>
                            <key>SensorOption</key>
                            <integer>1</integer>
                        </dict>
                        <key>control-id</key>
                        <integer>18</integer>
                    </dict>
                </dict>
            </dict>
        </dict>
    </dict>
    <key>OSBundleRequired</key>
    <string>Root</string>
</dict>
</plist>
```

---

## 2. IDT 92HD71B7 Headphone Jack Sensing (`alc-verb`)

If `AppleALC.kext` (Layout `11` or `12`) produces speaker audio but fails to switch when headphones are plugged into the 3.5mm jack, test forcing pin widget control on node `0x19` using `alc-verb`:

```bash
./alc-verb 0x19 0x707 0x20
```

---

## 3. Enabling SSD TRIM

To enable TRIM support on third-party SATA III SSDs in High Sierra:

```bash
sudo trimforce enable
```

*(Confirm the prompts; the system will reboot automatically. Expect an additional 15–30 seconds during boot on the ICH9M SATA controller while blocks are trimmed.)*

---

## 4. Legacy NVRAM Emulation (OpenCore `DuetPkg`)

Because the E6400 does not expose writable hardware NVRAM to macOS:

1. In `config.plist` under `NVRAM`:
   - Set `LegacyEnable` (`LegacyOverwrite` on newer OC) = `True`
   - Set `WriteFlash` = `True`
2. Place `OpenVariableRuntimeDxe.efi` in `EFI/OC/Drivers` (if using OpenCore `0.8.8+`) or install OpenCore's `LogoutHook` launch daemon so `nvram.plist` is written to the EFI partition on every clean shutdown/reboot.
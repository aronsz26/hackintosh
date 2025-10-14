# Hackintosh – Ryzen 5 2600X · MSI B350M PRO-VDH · Sapphire RX 460 4GB

Dieses Repository beschreibt Schritt für Schritt den Aufbau eines stabilen Hackintosh-Systems mit OpenCore auf folgender Hardware.

## Hardware

* **CPU:** AMD Ryzen 5 2600X (Zen+)
* **Mainboard:** MSI B350M PRO-VDH (MS-7A38)
* **GPU (primär empfohlen):** Sapphire **RX 460 4GB** (Polaris, GCN 4.0)
* **Alternative GPU:** AMD **RX 570 / RX 580** (ebenfalls nativ)
* **RAM:** DDR4 (≥ 8 GB)
* **Storage:** SATA/NVMe, **AHCI** im BIOS
* **LAN:** Realtek RTL8111
* **Audio:** Onboard Realtek ALC (per AppleALC)
* **Bootloader-USB:** ≥ 16 GB

> NVIDIA Pascal (GTX 1050 Ti) funktioniert **nur** unter macOS **High Sierra 10.13.x** mit **WebDriver** und ist für neuere macOS-Versionen nicht geeignet.

---

## Unterstützte macOS-Versionen

| Version               | RX 460 / RX 570/580  | R7 260X                | HD 6870        |
| --------------------- | -------------------- | ---------------------- | -------------- |
| High Sierra (10.13.6) | ✅ nativ (ab 10.13.4) | ✅                      | ✅              |
| Mojave (10.14)        | ✅                    | ⚠️ (nur eingeschränkt) | ❌ (kein Metal) |
| Catalina (10.15)      | ✅                    | ⚠️/**No** stabil       | ❌              |
| Big Sur (11)          | ✅                    | ❌                      | ❌              |
| Monterey (12)         | ✅                    | ❌                      | ❌              |
| Ventura (13)          | ✅                    | ❌                      | ❌              |
| **Sonoma (14)**       | ✅                    | ❌                      | ❌              |
| **Sequoia (15)**      | ✅                    | ❌                      | ❌              |

Empfehlung: **Monterey/Ventura/Sonoma/Sequoia** mit **RX 460/570/580**.
High Sierra nur, wenn unbedingt NVIDIA (WebDriver) benötigt wird.

---

## BIOS-Einstellungen (MSI Click BIOS 5)

* **Boot mode**: **UEFI**
* **Secure Boot**: **Disabled**
* **Fast Boot**: **Disabled**
* **CSM**: **Disabled**
* **SATA Mode**: **AHCI**
* **Above 4G Decoding**: **Enabled**
* **XHCI Hand-off**: **Enabled**
* **Primary Display**: PEG/PCIe (bei dGPU)

Speichern (F10).

---

## USB-Installer erstellen (Beispiel: Sonoma/Sequoia oder Monterey)

**Auf einem Mac:**

1. USB im Festplattendienstprogramm löschen: **GUID**, **Mac OS Extended (Journaled)**, Name `Untitled`.
2. Installer schreiben (Beispiel Monterey 12):

   ```bash
   sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --nointeraction
   ```

   (entspr. App für Sonoma/Sequoia verwenden)

Ergebnis: USB mit **EFI (200 MB)** und **Install macOS …**.

---

## OpenCore-EFI (Ordnerstruktur)

```
EFI
└─ OC
   ├─ ACPI
   │  ├─ SSDT-EC-USBX.aml
   │  └─ (optional) SSDT-AWAC.aml
   ├─ Drivers
   │  ├─ OpenRuntime.efi
   │  └─ OpenCanopy.efi   # optional GUI
   ├─ Kexts
   │  ├─ Lilu.kext
   │  ├─ WhateverGreen.kext
   │  ├─ VirtualSMC.kext
   │  ├─ AppleALC.kext
   │  ├─ RealtekRTL8111.kext
   │  ├─ NVMeFix.kext
   │  └─ USBToolBox.kext
   ├─ Resources           # optional (Themes)
   └─ config.plist
```

**Kext-Ladereihenfolge (Kernel → Add):**

```
Lilu
VirtualSMC
SMCProcessor (optional)
SMCSuperIO (optional)
WhateverGreen
AppleALC
RealtekRTL8111
NVMeFix
USBToolBox
```

---

## SMBIOS & Boot-Args

**SMBIOS (PlatformInfo → Generic):**

* Für AMD + dGPU: **iMacPro1,1** (empfohlen)
* Serien/MLB/UUID mit **GenSMBIOS** generieren (einzigartig).

**Boot-Args (NVRAM → 7C436110… → boot-args):**

* Für AMD + RX 460/570/580:

  ```
  agdpmod=pikera -v keepsyms=1 debug=0x100
  ```
* **Kein** `nv_disable=1` bei AMD.
* High Sierra + NVIDIA (WebDriver): erst `nv_disable=1`, nach Treiberinstallation `nvda_drv=1`.

---

## AMD (Ryzen) – Kernel-Patches (wichtig)

**Kernel → Patch**: komplettes Patch-Set für **Zen+** passend zur macOS-Version (Monterey/Ventura/Sonoma/Sequoia **oder** High Sierra) eintragen.
Zusätzlich empfohlene Quirks (Kernel → Quirks):

* `ProvideCurrentCpuInfo = True`
* `PanicNoKextDump = True`
* `DisableIoMapper = True`
* `DevirtualiseMmio = True`
* `XhciPortLimit = True` (nur für Installation/Erststart; später USB-Mapping erstellen)

---

## Installation (Kurzfassung)

1. **Vom USB (UEFI)** in **OpenCore-Picker** booten.
2. **Install macOS …** auswählen.
3. Im **Festplattendienstprogramm**: Ziel-SSD → **Löschen** (GUID, APFS).
4. Installation starten; bei Reboots im Picker den **Install/SSD-Eintrag** wählen, bis Setup startet.
5. Nach Ersteinrichtung: **EFI vom USB** auf **EFI der SSD** kopieren (damit ohne Stick gebootet wird).

---

## Post-Install

* **USB-Mapping:** mit **USBToolBox** ein **UTBMap.kext** erzeugen → stabiler Sleep/Wake.
* **Audio:** `AppleALC.kext` + `alcid=<LayoutID>` (z. B. 1/7/11/13) in den boot-args testen.
* **Netzwerk:** `RealtekRTL8111.kext` (Onboard Realtek).
* **NVRAM Reset** (Picker-Menü) nach größeren EFI-Änderungen.

---

## Troubleshooting (häufige Punkte)

**Hänger bei „End RandomSeed“ (Installer)**

* USB-Port wechseln (hinten, USB 2.0), `XhciPortLimit=True`.
* Prüfen: vollständiges **AMD-Patchset** für **korrekte macOS-Version**.
* SMBIOS korrekt? (iMacPro1,1 empfohlen).
* Bei NVIDIA/High Sierra: zunächst **`nv_disable=1`** benutzen.

**Schwarzer Bildschirm nach Boot**

* Bei Polaris/Navi: **`agdpmod=pikera`** in den boot-args.
* Prüfen, ob **WhateverGreen.kext** geladen wird.

**Kein Ton**

* `alcid` variieren (1, 7, 11, 13 …), AppleALC geladen?

**Kein LAN**

* `RealtekRTL8111.kext` vorhanden + aktiviert?

**OpenCore startet, aber macOS nicht**

* `SecureBootModel = Disabled` (Misc → Security)
* Quirks wie oben gesetzt.

---

## High Sierra + NVIDIA (optional, nur wenn nötig)

* Installer normal starten mit:

  ```
  nv_disable=1 -v keepsyms=1 debug=0x100 agdpmod=vit9696
  ```
* Nach macOS-Install **exakte** WebDriver-Version zur Build-Nummer installieren.
* Danach boot-args auf:

  ```
  nvda_drv=1 -v keepsyms=1 debug=0x100 agdpmod=vit9696
  ```

---

## Nützliche Projekte (Upstream)

* Acidanthera Kexts: Lilu, WhateverGreen, VirtualSMC, AppleALC, NVMeFix
* AMD Vanilla (Ryzen Patches)
* USBToolBox (USB-Mapping)
* GenSMBIOS (SMBIOS-Daten generieren)
* OpenCore (Bootloader)

*(Suche jeweils das offizielle GitHub-Repo der genannten Projekte.)*

---

## Haftungsausschluss

Dies ist eine technische Dokumentation zu Bildungszwecken. macOS auf Nicht-Apple-Hardware zu betreiben kann gegen Lizenzbedingungen verstoßen. Nutzung auf eigenes Risiko.

---

## Kurz-Checkliste

* [ ] BIOS wie oben gesetzt
* [ ] USB-Installer per `createinstallmedia` gebaut
* [ ] EFI (OC, Kexts, ACPI, config.plist) auf **EFI-Partition**
* [ ] SMBIOS iMacPro1,1 + eindeutige Serien/MLB/UUID
* [ ] Boot-Args korrekt (`agdpmod=pikera`, Debug-Flags)
* [ ] AMD-Patches (richtige macOS-Version!)
* [ ] Installation durchgelaufen, EFI auf SSD kopiert
* [ ] USB-Mapping, Audio, LAN geprüft

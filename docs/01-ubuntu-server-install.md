## [2026-10-27 / Step 01] — Ubuntu Server installation on Toshiba NB520

### 1. What was done
- Installed Ubuntu Server 26.04.1 LTS (amd64) on the internal 1 TB WDC WD10SPZX disk from a USB flash drive.
- Resolved two boot/install failures (GRUB rescue, installer not seeing the disk).
- Extended the root logical volume from 100 GB to all free space in the volume group.
- Result: system boots from the internal disk and runs fine offline. No network yet.

### 2. Configuration / Commands
```bash
# Extend root LV to all free space and resize the ext4 filesystem in one step
lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

```text
# Windows laptop, to wipe the leftover installer image from the 1 TB disk
diskpart
  select disk <N>
  clean
```

Resulting layout:

| Device | Size | Role |
|--------|------|------|
| `sda1` | 1 MB | (role not recorded) |
| `sda2` | 2 GB, ext4 | `/boot` |
| `sda3` | 929.5 GB | LVM PV (`ubuntu-vg`) |
| `ubuntu-vg/ubuntu-lv` | all of `sda3` | ext4 `/` |

Host: Toshiba NB520, Intel Atom N570 (4 threads), 1.6 GiB RAM, 2 GiB swap, kernel 7.0.0-30-generic, hostname `vovaserver`, user `vova`.

### 3. Errors and fixes

#### 3.1 `grub rescue>` — `symbol 'grub_memcpy' not found`
- **When:** 2026-10-15, booting the disk written with Rufus.
- **Cause:** Rufus in ISO Image mode wrote a GRUB that did not match the one on the ISO.
- **Fix:** Rewrite the media in Rufus **DD Image** mode.

#### 3.2 Installer shows no disks; `wipefs` on `sda` fails with "Device or resource busy"
- **When:** 2026-10-27.
- **Cause:** The installer image had been left on the 1 TB disk (iso9660) and `/cdrom` was mounted from that disk, so the installer could not use or wipe it.
- **Fix:** Cleaned the disk with `diskpart clean` on a Windows laptop, then installed from a separate USB flash drive.

### 4. Theory
- **Rufus modes:** ISO Image mode rebuilds the boot structure (including its own bootloader files); DD Image mode writes the ISO byte-for-byte. For a hybrid Ubuntu ISO, DD avoids bootloader mismatches.
- **LVM:** `sda3` is a physical volume in `ubuntu-vg`; the installer by default creates `ubuntu-lv` smaller than the disk. `lvextend -r` grows the LV and resizes the filesystem (`-r`) without unmounting.
- **iso9660 on the target disk:** a disk that still holds an installer image keeps its filesystem signature and can be mounted as the live medium, which makes it busy for `wipefs` and hidden from the installer.

### 5. Open items
- No network: `enp9s0` (Ethernet) and `wlp7s0` (Wi-Fi) are DOWN, no routes.
- UFW inactive; SSH on port 22 with password auth enabled, root login `prohibit-password`.
- Next: bring up networking, then SSH hardening, UFW, Tailscale.

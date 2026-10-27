# Server State

Single source of truth for the home server. Keep it short and current.
Update after every documented stage. Repo: https://github.com/VolodyaShchyrov/Laptop_Linux_Ubuntu_Server

## Hardware / OS
- Host: Toshiba NB520 netbook (Intel Atom N570, 4 threads, 1.6 GiB RAM, 2 GiB swap, firmware V1.90 from 2011)
- OS: Ubuntu Server 26.04.1 LTS (amd64)
- Kernel: 7.0.0-30-generic
- Hostname: vovaserver
- Disks / partitions: WDC WD10SPZX 931.5 GB (`sda`, SATA, internal, system disk)
  - sda1 1 MB, sda2 2 GB ext4 `/boot`, sda3 929.5 GB LVM
  - LVM: `ubuntu-vg/ubuntu-lv` 100 GB ext4 `/` (rest of the volume group not yet allocated)

## Network
- LAN IP (static/DHCP): none yet (no connection)
- Gateway / DNS: none (no routes)
- Interfaces: `enp9s0` (Ethernet) DOWN, `wlp7s0` (Wi-Fi) DOWN
- Tailscale IP / tailnet name: TBD
- Open ports (UFW): UFW inactive

## Access
- SSH: port 22, key-only auth: no (password auth enabled), root login: prohibit-password
- Users: vova

## Services
| Service | Method (systemd/Docker) | Port | Status |
|---------|-------------------------|------|--------|
| TBD     | TBD                     | TBD  | TBD    |

## Done
- Stage 1: Ubuntu Server 26.04.1 installed on the internal 1 TB WDC disk (installed from USB flash drive); system boots from the disk and runs fine offline

## Known issues / open questions
- Server has no network/internet yet; Ethernet and Wi-Fi interfaces are DOWN
- Root LV is only 100 GB of 929.5 GB; extend with `lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv`
- UFW is inactive
- SSH allows password login

## Error log (short, details live in docs/)
| Date | Symptom | Cause | Fix |
|------|---------|-------|-----|
| 2026-10-15 | `grub rescue>`, `symbol 'grub_memcpy' not found` when booting the Rufus-written disk | Rufus ISO Image mode wrote a GRUB that mismatched the ISO | Rewrite in Rufus DD Image mode |
| 2026-10-27 | Installer shows no disks; `wipefs` on `sda` fails with "Device or resource busy" | Installer image was left on the 1 TB disk (iso9660) and `/cdrom` was mounted from it | Cleaned the disk with `diskpart clean` on a Windows laptop, then installed from a USB flash drive |

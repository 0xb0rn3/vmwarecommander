# vmwarecommander

```
╦  ╦╔╦╗╦ ╦╔═╗╦═╗╔═╗  ╔═╗╔═╗╔╦╗╔╦╗╔═╗╔╗╔╔╦╗╔═╗╦═╗
╚╗╔╝║║║║║║╠═╣╠╦╝║╣   ║  ║ ║║║║║║║╠═╣║║║ ║║║╣ ╠╦╝
 ╚╝ ╩ ╩╚╩╝╩ ╩╩╚═╚═╝  ╚═╝╚═╝╩ ╩╩ ╩╩ ╩╝╚╝═╩╝╚═╝╩╚═
```

**Automated VMware Workstation installation and maintenance across Linux distributions**

Built to eliminate the pain of manual VMware setup after kernel updates. Handles dependency resolution, AUR package building, kernel module compilation, and automatic DKMS rebuilds.

---

## What it does

- Detects your kernel (standard, LTS, zen, hardened)
- Installs dependencies from official repositories
- Builds AUR packages with proper user permissions (no root makepkg errors)
- Detects or installs AUR helpers (yay/paru) with optional cleanup
- Falls back to manual `makepkg -si` if AUR helpers fail
- Loads kernel modules (vmmon, vmw_vmci, vmnet)
- Configures systemd services
- Creates distribution-specific hooks for automatic DKMS rebuilds on kernel updates

## Supported distributions

**Arch-based**: Arch Linux, ArchBANG, Manjaro, EndeavourOS, Garuda, ArcoLinux, CachyOS, Artix, Crystal Linux

**Debian-based**: Debian, Ubuntu, Linux Mint, Pop!_OS, Elementary, Zorin, Kali, Parrot, MX Linux

**Fedora-based**: Fedora, RHEL, CentOS, Rocky Linux, AlmaLinux, Nobara, Ultramarine

**openSUSE**: openSUSE Leap, Tumbleweed, SLES

**Others**: Void Linux

## Installation

### One-liner

```bash
curl -fsSL https://raw.githubusercontent.com/0xb0rn3/vmwarecommander/main/vmwarecommander | sudo bash
```

### Manual

```bash
git clone https://github.com/0xb0rn3/vmwarecommander.git
cd vmwarecommander
chmod +x vmwarecommander
sudo ./vmwarecommander
```

The script handles everything else.

## Requirements

- Active internet connection
- sudo/root access
- For Arch-based: kernel headers matching your kernel version
- For other distros: VMware Workstation bundle (downloaded separately)

## How automatic rebuilds work

The script creates distribution-specific hooks:

**Arch Linux**: `/etc/pacman.d/hooks/90-vmware-dkms.hook`  
When you run `pacman -Syu` and a new kernel gets installed, this hook triggers `dkms autoinstall` automatically before you reboot. No more broken VMs after updates.

**Debian/Ubuntu**: `/etc/apt/apt.conf.d/99-vmware-dkms`  
Triggers on kernel package installation/upgrade.

**Fedora/RHEL**: `/etc/dnf/plugins/post-transaction-actions.d/vmware-dkms.action`  
Runs after DNF transactions involving kernel packages.

**openSUSE**: Systemd service for DKMS rebuilds.

## AUR helper handling

The script intelligently manages AUR helpers on Arch-based systems:

1. Checks for existing yay or paru
2. If none found, offers to install yay temporarily
3. Uses the helper to install vmware-workstation
4. If AUR helper fails, falls back to manual `makepkg -si`
5. After installation, asks if you want to keep or remove the temporary helper

All builds run as your regular user (not root) to avoid the infamous "Running makepkg as root is not allowed" error.

## Tested on

- ArchBANG with kernel 6.18.3-arch1-1
- Arch Linux (standard, LTS, zen kernels)
- Manjaro, EndeavourOS, Garuda
- Ubuntu 24.04, Debian 12
- Fedora 40

## Troubleshooting

**VMware won't start after kernel update**

The hook should handle this automatically, but if modules aren't loading:

```bash
sudo dkms autoinstall
sudo modprobe -a vmw_vmci vmmon vmnet
```

**Check if modules compiled**

```bash
lsmod | grep vmmon
lsmod | grep vmw_vmci
dkms status
```

**Services not running**

```bash
systemctl status vmware-networks.service
systemctl status vmware-usbarbitrator.service
```

Restart them:

```bash
sudo systemctl restart vmware-networks.service vmware-usbarbitrator.service
```

**AUR build fails**

The script will automatically fall back to manual building. If that also fails:

```bash
cd ~/.cache/aur-build/vmware-workstation
makepkg -si
```

## Manual installation (if you prefer to see what happens)

```bash
# Install kernel headers
sudo pacman -S linux-headers

# Install dependencies
sudo pacman -S base-devel dkms libxcrypt-compat fuse2 gtkmm3 libcanberra pcsclite git gcc make

# Build from AUR (as regular user, not root)
git clone https://aur.archlinux.org/vmware-workstation.git
cd vmware-workstation
makepkg -si

# Load modules
sudo modprobe -a vmw_vmci vmmon vmnet

# Enable services
sudo systemctl enable --now vmware-networks.service
sudo systemctl enable --now vmware-usbarbitrator.service
```

## Why this exists

Manually rebuilding VMware modules after every kernel update is tedious. The AUR package works great, but the dependency chain and build process trips people up. Common issues:

- Running makepkg as root (forbidden)
- Missing AUR helpers
- Kernel header mismatches
- Forgetting to rebuild modules after updates

This tool automates all of it and sets up persistent maintenance so your VMs keep working after kernel updates.

## License

MIT

## Credits

**Developer**: 0xb0rn3 | oxbv1  
**Version**: 1.1.0

---

If this saved you time, drop a ⭐

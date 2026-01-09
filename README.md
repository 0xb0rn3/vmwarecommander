# vmwarecommander

```
╦  ╦╔╦╗╦ ╦╔═╗╦═╗╔═╗  ╔═╗╔═╗╔╦╗╔╦╗╔═╗╔╗╔╔╦╗╔═╗╦═╗
╚╗╔╝║║║║║║╠═╣╠╦╝║╣   ║  ║ ║║║║║║║╠═╣║║║ ║║║╣ ╠╦╝
 ╚╝ ╩ ╩╚╩╝╩ ╩╩╚═╚═╝  ╚═╝╚═╝╩ ╩╩ ╩╩ ╩╝╚╝═╩╝╚═╝╩╚═
```

**Automated VMware Workstation installation and maintenance for Arch Linux**

Built out of frustration with manual VMware setup after every kernel update. This tool handles the entire process from dependency resolution to automatic kernel module rebuilds.

---

## What it does

- Detects your kernel type (standard, LTS, zen, hardened)
- Installs all required dependencies from official repos
- Builds and installs AUR packages (vmware-keymaps, vmware-workstation)
- Loads kernel modules (vmmon, vmw_vmci)
- Configures systemd services for networking and USB passthrough
- Creates a Pacman hook that rebuilds DKMS modules automatically on kernel updates

## Installation
## Oneliner 
```bash
 curl -fsSL https://raw.githubusercontent.com/0xb0rn3/vmwarecommander/main/vmwarecommander | sudo bash
```
## OR MANUALLY
```bash
git clone https://github.com/0xb0rn3/vmwarecommander.git
cd vmwarecommander
chmod +x vmwarecommander
sudo ./vmwarecommander
```

That's it. The script handles everything else.

## Requirements

- Arch Linux (tested on ArchBANG)
- Active internet connection
- sudo/root access

## How the Pacman hook works

When you run `pacman -Syu` and a new kernel gets installed, the hook at `/etc/pacman.d/hooks/90-vmware-dkms.hook` automatically triggers `dkms autoinstall`. This rebuilds VMware's kernel modules before you even reboot.

No more broken VMs after updates.

## Tested on

- ArchBANG with kernel 6.18.3-arch1-1
- Standard Arch kernel
- Intel i7-8665U CPU

## Troubleshooting

**VMware won't start after kernel update**

The hook should handle this automatically, but if something breaks:
```bash
sudo dkms autoinstall
sudo modprobe -a vmw_vmci vmmon
```

**Modules not loading**

Check if they compiled:
```bash
lsmod | grep vmmon
lsmod | grep vmw_vmci
```

If nothing shows up:
```bash
sudo dkms status
```

**Services not running**

```bash
sudo systemctl status vmware-networks.service
sudo systemctl status vmware-usbarbitrator.service
```

Restart them:
```bash
sudo systemctl restart vmware-networks.service vmware-usbarbitrator.service
```

## Manual installation (if you want to see what happens)

```bash
# Install kernel headers
sudo pacman -S linux-headers  # or linux-lts-headers

# Install dependencies
sudo pacman -S base-devel dkms libxcrypt-compat libxml2

# Build AUR packages
git clone https://aur.archlinux.org/vmware-keymaps.git
cd vmware-keymaps && makepkg -si && cd ..

git clone https://aur.archlinux.org/vmware-workstation.git
cd vmware-workstation && makepkg -si && cd ..

# Load modules
sudo modprobe -a vmw_vmci vmmon

# Enable services
sudo systemctl enable --now vmware-networks.service
sudo systemctl enable --now vmware-usbarbitrator.service
```

## Why this exists

Because manually rebuilding VMware modules after every kernel update is tedious. The AUR package is great, but the dependency chain (vmware-keymaps → libxml2-legacy → vmware-workstation) trips people up. This tool automates all of it and sets up persistent maintenance.

## License

MIT

## Credits

**Developer:** 0xb0rn3 | oxbv1  
**Version:** 0.0.1

---

If this saved you time, drop a ⭐

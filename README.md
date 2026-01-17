# vmwarecommander

```
╦  ╦╔╦╗╦ ╦╔═╗╦═╗╔═╗  ╔═╗╔═╗╔╦╗╔╦╗╔═╗╔╗╔╔╦╗╔═╗╦═╗
╚╗╔╝║║║║║║╠═╣╠╦╝║╣   ║  ║ ║║║║║║║╠═╣║║║ ║║║╣ ╠╦╝
 ╚╝ ╩ ╩╚╩╝╩ ╩╩╚═╚═╝  ╚═╝╚═╝╩ ╩╩ ╩╩ ╩╝╚╝═╩╝╚═╝╩╚═
```

**Intelligent VMware Workstation installer with detection, repair, and automatic maintenance for Arch Linux**

Eliminates manual VMware setup headaches with smart detection, three installation modes, and automatic kernel module rebuilds after updates.

-----

## Features

✨ **Smart Detection** - Automatically detects existing VMware installations and their health status  
🔧 **Three Installation Modes** - Fresh install, repair, or skip based on current state  
🚀 **Fully Automated** - Handles AUR packages, dependencies, modules, and services  
🔄 **Auto-Rebuild** - Pacman hook rebuilds modules automatically on kernel updates  
🎯 **Zero Manual Configuration** - Network setup, groups, services all automated  
📊 **Comprehensive Verification** - Detailed status checks post-installation

## What it does

### Installation Modes

**1. Fresh Install**

- Detects and removes existing VMware installation
- Cleans configs, modules, and services
- Performs complete reinstallation
- Perfect for corrupted installations or major version upgrades

**2. Repair Mode**

- Keeps existing installation and VM configurations
- Rebuilds kernel modules
- Restarts services
- Fixes common issues without data loss
- Ideal for post-kernel-update problems

**3. Intelligent Detection**
The script automatically checks:

- VMware binary presence and version
- Package installation status
- Kernel module loading state
- Service running status
- Then recommends the best action

### Core Capabilities

- Detects kernel type (standard, LTS, zen, hardened)
- Installs all required dependencies
- Builds AUR packages (`vmware-keymaps`, `vmware-workstation`)
- Manages AUR helpers (detects yay/paru, installs temporarily if needed)
- Configures kernel modules (vmmon, vmw_vmci, vmnet)
- Sets up systemd services with auto-start
- Creates VMware user group and permissions
- Configures virtual networks (vmnet1, vmnet8)
- Installs pacman hook for automatic module rebuilds

## Supported Systems

**Arch-based distributions**: Arch Linux, ArchBANG, Manjaro, EndeavourOS, Garuda, ArcoLinux, CachyOS, Artix, Crystal Linux, BlackArch

## Installation

```bash
git clone https://github.com/0xb0rn3/vmwarecommander.git
cd vmwarecommander
chmod +x vmwarecommander
sudo ./vmwarecommander
```

The script handles everything else automatically.

## Requirements

- Active internet connection
- sudo/root access
- Kernel headers matching your kernel version
- ~5GB free disk space for build process

## How it works

### Detection Phase

When you run the script, it first analyzes your system:

```
[*] Current VMware Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Binary: VMware Workstation 17.5.0 build-22583795
✓ Package: vmware-workstation 17.5.0-1
✗ Modules: Not loaded
✓ Services: vmware-networks.service vmware-usbarbitrator.service
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠ Found 1 issue(s) with current installation
```

### Interactive Mode Selection

Based on detection, you choose:

```
Choose installation mode:
  [1] Fresh Install (remove and reinstall everything)
  [2] Repair (fix modules and services, keep config)
  [3] Skip (exit without changes)
```

### Automatic Rebuilds

The script creates `/etc/pacman.d/hooks/90-vmware-dkms.hook`:

```ini
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = linux
Target = linux-lts
Target = linux-zen
Target = linux-hardened
Target = vmware-workstation

[Action]
Description = Rebuilding VMware kernel modules
When = PostTransaction
Exec = /usr/bin/vmware-modconfig --console --install-all
```

When you run `pacman -Syu` and a kernel update is installed, modules are automatically rebuilt **before** you reboot. No more broken VMs after system updates.

### Network Configuration

Automatically configures two virtual networks:

- **vmnet1 (Host-Only)**: 192.168.100.0/24 with DHCP
- **vmnet8 (NAT)**: 192.168.200.0/24 with DHCP and NAT

VMs get network access immediately without manual setup.

## AUR Helper Management

The script intelligently handles AUR helpers:

1. Detects existing `yay` or `paru`
1. If none found, offers to install `yay` temporarily
1. Builds VMware packages as non-root user (avoids makepkg errors)
1. After installation, asks if you want to keep or remove the temporary helper
1. Falls back to manual `makepkg -si` if AUR helper fails

All package builds run as your regular user to comply with AUR security requirements.

## Usage Examples

### First-time installation

```bash
sudo ./vmwarecommander
# Script detects no VMware, performs fresh install
# Sets up everything automatically
```

### After kernel update (modules not loading)

```bash
sudo ./vmwarecommander
# Detects existing installation with module issues
# Choose [2] Repair
# Rebuilds modules, restarts services
# VMs work again
```

### Complete reinstall (corruption or version upgrade)

```bash
sudo ./vmwarecommander
# Detects problematic installation
# Choose [1] Fresh Install
# Removes everything cleanly, reinstalls from scratch
```

## Post-Installation

### Verification

The script performs comprehensive checks:

```
[*] Installation verification
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ vmmon kernel module loaded
✓ vmw_vmci kernel module loaded
✓ vmnet kernel module loaded
✓ VMware binary: VMware Workstation 17.5.0
✓ vmrun utility installed
✓ vmware-keymaps installed
✓ vmware-workstation 17.5.0-1 installed
✓ vmware-networks.service enabled and running
✓ vmware-usbarbitrator.service enabled and running
✓ User in vmware group
✓ VMware networking configured
✓ Module auto-load configured
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Launch VMware

```bash
# GUI
vmware &

# Command line VM management
vmrun list
vmrun start /path/to/vm.vmx

# Check status
systemctl status vmware-networks
lsmod | grep vm
```

### If warnings appear

Common warning after installation:

```
⚠ 2 warning(s) detected
Required actions:
  1. Reboot system: sudo reboot
  2. After reboot, log out and back in for group changes
```

This is normal for first-time installations. Reboot to activate all changes.

## Troubleshooting

### Modules not loading after kernel update

The pacman hook should handle this automatically. If it doesn’t:

```bash
sudo vmware-modconfig --console --install-all
sudo modprobe vmmon vmw_vmci vmnet
```

### Services not starting

```bash
sudo systemctl daemon-reload
sudo systemctl restart vmware-networks
sudo systemctl restart vmware-usbarbitrator
```

### Check module compilation

```bash
lsmod | grep vm
dkms status
```

### VMware UI won’t start

```bash
# Check if binary exists
which vmware

# Try running with verbose output
vmware -v

# Check logs
journalctl -xe | grep vmware
```

### Network issues in VMs

```bash
# Restart network services
sudo systemctl restart vmware-networks

# Check virtual networks
sudo vmware-networks --status

# Manually start networks
sudo vmware-networks --start
```

### AUR build fails

If the script’s automatic builds fail, manual fallback:

```bash
cd ~/.cache/aur-build/vmware-workstation
makepkg -si
```

Or let the script retry - it has multiple fallback mechanisms.

### Repair vs Fresh Install

**Choose Repair when:**

- Modules won’t load after kernel update
- Services stopped working
- You want to keep VM configurations
- Minor issues, installation mostly working

**Choose Fresh Install when:**

- VMware completely broken
- Upgrading between major versions
- Corruption suspected
- Want to start completely clean

## What gets installed

### Packages

**From official repos:**

- base-devel, linux-headers, dkms, git
- fuse2, gtkmm3, libcanberra, pcsclite
- gcc, make, net-tools, inetutils
- libxcrypt-compat, ncurses5-compat-libs
- usbutils, wget, unzip

**From AUR:**

- vmware-keymaps
- vmware-workstation

### Files created/modified

```
/etc/pacman.d/hooks/90-vmware-dkms.hook    # Auto-rebuild hook
/etc/modules-load.d/vmware.conf            # Module auto-load
/etc/vmware/networking                     # Network config
/etc/init.d/vmware                         # Init script (if applicable)
/usr/lib/vmware/                           # VMware installation
```

### Services

```
vmware-networks.service        # Virtual network management
vmware-usbarbitrator.service   # USB device passthrough
vmware-hostd.service           # Host daemon (if installed)
```

## Tested on

- ArchBANG with KDE Plasma (X11)
- Arch Linux (standard, LTS, zen, hardened kernels)
- Manjaro, EndeavourOS, Garuda
- BlackArch
- CachyOS

## Why this exists

**The problem:**

- VMware modules break after every kernel update
- Manual rebuilding is tedious and error-prone
- AUR packages trip up new users
- “Running makepkg as root” errors are confusing
- Reinstalling means losing all VM configs

**The solution:**

- Automatic detection of installation state
- Repair mode preserves configurations
- Pacman hook rebuilds modules automatically
- AUR helper management done right
- Complete automation with verification

One script, three modes, zero headaches.

## Version History

**v2.1.0** (Current)

- Added intelligent VMware detection
- Three installation modes (fresh/repair/skip)
- Enhanced status reporting
- Improved error handling
- Better user interaction

**v1.1.0**

- Multi-distro support
- DKMS auto-rebuild hooks
- Manual build fallbacks

**v1.0.0**

- Initial Arch Linux support

## License

MIT License - See LICENSE file

## Credits

**Developer**: 0xb0rn3 | oxbv1  
**Version**: 2.1.0  
**Repository**: https://github.com/0xb0rn3/vmwarecommander

-----

## Contributing

Found a bug? Have a suggestion?

1. Open an issue with details
1. Include your distro, kernel version, and logs from `/tmp/vmwarecommander-*.log`
1. PRs welcome for bug fixes and improvements

-----

If this saved you time debugging VMware after a kernel update, drop a ⭐

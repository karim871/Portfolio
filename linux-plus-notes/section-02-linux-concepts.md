# CompTIA Linux+ Study Guide
## Section 2 — Linux Concepts (Domain 1.1: System Management)

---

> **Exam objective:** Explain basic Linux concepts.
> **Weight:** High — this section underpins everything else in the course.

---

## 1. Linux Overview

### What is Linux?
Linux is an **open-source operating system** inspired by (but not derived from) UNIX.

| Property | UNIX | Linux |
|---|---|---|
| Created | Late 1960s | 1991 (Linus Torvalds) |
| License | Proprietary | Free & Open Source |
| Source code | Closed | Publicly available |
| Cost | Paid | Free |

### CLI vs GUI — Why admins prefer the terminal
- **Faster** — no rendering overhead
- **Less resource-intensive** — runs on minimal hardware
- **Scriptable** — automate repetitive tasks
- **More flexible** — fine-grained control over every parameter

> **Exam tip:** Linux admins favor the CLI not because there is no GUI, but because the CLI is more powerful and automatable. Many modern distros (Ubuntu, Fedora) ship with full GUIs.

---

## 2. The Linux Boot Process

### Big Picture — Boot Sequence (in order)

```
Power ON
   ↓
Firmware (BIOS / UEFI)
   ↓
[Optional] PXE (network boot) ──────────────────────┐
   ↓                                                 │
GRUB2 (Bootloader)  ←────────────────────────────────┘
   ↓
initrd (Initial RAM Disc) + Linux Kernel  ← loaded simultaneously
   ↓
Kernel initializes hardware → mounts real root filesystem
   ↓
System processes start → OS fully operational
```

---

### Component 1: PXE — Preboot Execution Environment

**What it is:** Technology that lets a computer boot over a **network** instead of local storage (HDD/SSD).

**How it works:**
1. NIC with PXE support powers on
2. NIC contacts a **DHCP server** → gets an IP address + TFTP server location
3. NIC contacts a **TFTP server** (Trivial File Transfer Protocol) → downloads boot files (e.g., GRUB2)
4. GRUB2 loads an OS installer or pre-configured image

**Use case:** Deploying Linux to 50 computers in a school lab from one central server — no USB drives, no manual installs.

> **Exam keywords:** PXE, DHCP, TFTP, network boot, centralized deployment

---

### Component 2: GRUB2 — Grand Unified Bootloader version 2

**What it does:** Selects and launches the Linux kernel.

**Key files:**
| File | Purpose |
|---|---|
| `/boot/grub/grub.cfg` | Auto-generated config (boot entries, kernel versions, parameters) |
| `/etc/default/grub` | Admin-editable settings file |

**Key command:**
```bash
update-grub        # Regenerates grub.cfg after editing /etc/default/grub
```

**Flow:**
1. Firmware (BIOS/UEFI) hands control to GRUB2
2. GRUB2 displays boot menu (if multiple kernels/OSes exist)
3. GRUB2 reads `grub.cfg`
4. GRUB2 loads **initrd** + **kernel**

> **Exam tip:** You edit `/etc/default/grub`, then run `update-grub` to apply changes. Never edit `grub.cfg` directly.

---

### Component 3: initrd — Initial RAM Disc

**What it is:** A **temporary root filesystem** loaded into RAM before the real root filesystem mounts.

**File location:** `/boot/initrd.img`

**Why it exists:** The kernel needs certain drivers (storage controllers, encryption modules, filesystem drivers) to even *find* the real root partition. The initrd provides those drivers before the full OS is available.

**Loaded:** Simultaneously with the kernel (not after).

**Real-world example:** Encrypted root partition → initrd contains the decryption module → unlocks disk → kernel mounts real filesystem.

**Admin commands:**
```bash
mkinitramfs    # Debian/Ubuntu — generate or update initrd
dracut         # RHEL/Fedora — generate or update initrd
```

> **Exam tip:** initrd = temporary → bridges the gap between bootloader and full OS. It is NOT permanent storage.

---

### Component 4: The Linux Kernel

**What it is:** The **core of the OS** — manages hardware, processes, memory, and system resources.

**What it does at boot:**
1. Initializes system hardware
2. Detects devices
3. Mounts the real root filesystem
4. Starts essential system processes

**Key tool: `sysctl`** — modify kernel parameters at runtime

```bash
# View/set a parameter at runtime
sysctl net.ipv4.ip_forward=1

# Apply saved settings without rebooting
sysctl -p
```

**Persistent kernel parameters file:** `/etc/sysctl.conf`

**Example — enable IP forwarding permanently:**
```bash
# Add to /etc/sysctl.conf:
net.ipv4.ip_forward=1

# Apply without reboot:
sysctl -p
```

> **Exam tip:** `sysctl` = runtime kernel tuning. `/etc/sysctl.conf` = persistent. `sysctl -p` = reload from file.

---

### Boot Process Summary Table

| Component | Role | Key File/Command |
|---|---|---|
| PXE | Network boot via DHCP + TFTP | NIC firmware |
| GRUB2 | Loads kernel + initrd, shows boot menu | `/boot/grub/grub.cfg`, `/etc/default/grub`, `update-grub` |
| initrd | Temporary filesystem, provides early drivers | `/boot/initrd.img`, `mkinitramfs`, `dracut` |
| Kernel | Core OS, initializes hardware, mounts root FS | `/etc/sysctl.conf`, `sysctl -p` |

---

## 3. System Directories (FHS)

The **File System Hierarchy Standard (FHS)** defines where things live in Linux. Everything begins at `/` (root).

### Critical System Directories

| Directory | Purpose | Key detail |
|---|---|---|
| `/` | Root — top of the entire filesystem | Everything lives under here |
| `/bin` | Essential user binaries (available to all users) | Commands like `ls`, `cp`, `mv` |
| `/sbin` | Essential system/admin binaries | Commands like `fdisk`, `ifconfig` |
| `/boot` | Boot files | `grub.cfg`, `initrd.img`, kernel images |
| `/dev` | Device files | Hardware represented as files (e.g., `/dev/sda`) |
| `/etc` | System-wide configuration files | Text-based config — never binaries |
| `/lib` | Shared libraries needed by `/bin` and `/sbin` | Like `.dll` files in Windows |
| `/media` | Mount points for removable media | USB drives, CDs |
| `/mnt` | Temporary mount points (admin-created) | Manual mounts |
| `/opt` | Optional/third-party software | Self-contained apps |
| `/proc` | Virtual filesystem — kernel/process info | Not real files; generated at runtime |
| `/root` | Home directory for the root user | Different from `/` |
| `/run` | Runtime data since last boot | PIDs, sockets |
| `/sys` | Virtual filesystem — hardware/kernel info | Similar to `/proc` |

> **Exam tip:** `/proc` and `/sys` are **virtual** — they contain no actual files on disk. They expose kernel data structures as readable files.

---

## 4. User and Application-Related Directories

These directories store **user data, installed apps, logs, and temporary files** — distinct from system directories.

### Windows vs Linux Comparison

| Windows | Linux Equivalent | Purpose |
|---|---|---|
| `C:\Users\` | `/home` | User personal files |
| `C:\Program Files` | `/usr` | Application binaries & libraries |
| `C:\Windows\System32` logs | `/var/log` | Log files |
| `%TEMP%` | `/tmp` | Temporary files |

---

### `/home` — User Home Directories

- Each user gets a subdirectory: `/home/username`
- Stores: documents, downloads, config files (dotfiles like `.bashrc`)
- Standard users have **full control** of their own `/home` — no root needed
- Root's home is **`/root`** (not under `/home`)

```bash
ls /home          # List all users with home directories
```

---

### `/usr` — User Programs and Application Data

One of the **largest directories** on a Linux system. Organized by subdirectory:

| Subdirectory | Contents |
|---|---|
| `/usr/bin` | Application binaries for regular users (text editors, browsers, compilers) |
| `/usr/sbin` | Admin tools requiring elevated privileges |
| `/usr/lib` | Shared libraries for `/usr/bin` and `/usr/sbin` apps |
| `/usr/share` | Documentation, icons, non-executable data |

> **Key distinction:** `/bin` = essential system binaries (needed to boot). `/usr/bin` = non-critical applications (installed after boot).

---

### `/var` — Variable Data

Holds **frequently changing** data — files here grow and shrink constantly.

| Subdirectory | Contents |
|---|---|
| `/var/log` | System and application logs |
| `/var/mail` | User mailboxes |
| `/var/www` | Website files (web server data) |
| `/var/cache` | Cached application data |

**Key property:** Files in `/var` are **retained until manually deleted** (unlike `/tmp`).

```bash
ls /var/log                    # List log files
cat /var/log/auth.log          # Read authentication log
```

---

### `/tmp` — Temporary Files

- Stores **short-term** data created by running applications
- Files are **automatically deleted** on reboot or after a period of inactivity
- Examples: session files, undo buffers while editing, cached downloads

> **Exam warning:** Never store important files in `/tmp` — they will be deleted automatically.

---

### Directory Summary

| Directory | Persistent? | Owner | Use |
|---|---|---|---|
| `/home` | Yes | User | Personal files |
| `/usr` | Yes | System | Installed applications |
| `/var` | Yes (until deleted) | System | Logs, mail, web data |
| `/tmp` | No (cleared on reboot) | Anyone | Temporary scratch space |

---

## 5. Server Architectures

**Architecture** = the design of a CPU and how it processes data. Choosing the right architecture affects performance, power consumption, and scalability.

### The Four Main Architectures

---

#### x86 (32-bit)

- Developed by **Intel**, older design
- Processes data in **32-bit chunks**
- Max RAM addressable: **4 GB** (2³² bytes)
- Used in: older PCs, legacy servers, embedded devices
- Status: **largely obsolete** for modern servers

> **Exam tip:** x86 = 32-bit = 4 GB RAM limit. That's its core weakness.

---

#### x86_64 / AMD64 (64-bit)

- Introduced by **AMD in 1999**, later adopted by Intel
- Processes data in **64-bit chunks**
- Theoretical max RAM: **16 exabytes** (2⁶⁴ bytes)
- Practical Linux limit: **~128 TB RAM** (enterprise hardware can go higher)
- Used in: **most modern Linux servers** — widely supported, broad application compatibility

> **Exam tip:** AMD64 = 64-bit extension of x86. Most common server architecture today.

---

#### AArch64 / ARM64 (64-bit ARM)

- 64-bit version of ARM (Advanced RISC Machine) processors
- Designed for **power efficiency and scalability**
- Used in: mobile devices, cloud servers, energy-conscious data centers
- Example hardware: **Raspberry Pi 4** running Ubuntu Server

> **Exam tip:** AArch64 = ARM64 = energy-efficient. Think cloud and mobile workloads.

---

#### RISC-V (Open-Source)

- **Reduced Instruction Set Computing — 5th major iteration**
- **Open-source CPU design** — free to use and modify (unlike x86 or ARM which are proprietary)
- Still emerging — growing in: research, embedded systems, AI accelerators, IoT, edge computing
- Key advantage: **fully customizable** processor design

> **Exam tip:** RISC-V = open source architecture. Its differentiator is that anyone can modify the design.

---

### Architecture Comparison Table

| Architecture | Bits | RAM Limit | Strengths | Use Case |
|---|---|---|---|---|
| x86 | 32 | 4 GB | Legacy compatibility | Old servers, embedded |
| x86_64 / AMD64 | 64 | ~128 TB | Performance, compatibility | Most modern servers |
| AArch64 / ARM64 | 64 | High | Power efficiency | Cloud, mobile, Pi |
| RISC-V | 64 | High | Open source, customizable | Research, AI, IoT |

---

## 6. Linux Distributions

**Linux kernel** (created 1991, Linus Torvalds) + utilities + libraries + package manager = **Linux Distribution (distro)**

### Two Main Categories

Linux distros split primarily on their **package management system**.

---

### RPM-Based Distributions

**RPM** = Red Hat Package Manager

| Distro | Notes |
|---|---|
| **RHEL** (Red Hat Enterprise Linux) | Commercial, paid subscription, enterprise-grade support |
| **Fedora** | RHEL's upstream testing ground — bleeding edge features |
| **CentOS Stream** | Rolling preview of RHEL (replaced classic CentOS) |
| **AlmaLinux / Rocky Linux** | Free, community-driven RHEL replacements |
| **openSUSE** | Uses `zypper` instead of `dnf` |

**Package format:** `.rpm`

**Package managers:**
```bash
dnf install firefox        # Fedora, RHEL, CentOS (formerly: yum)
yum install firefox        # Older RHEL/CentOS systems
zypper install firefox     # openSUSE only
sudo dnf install firefox   # with root privileges
```

> **Exam tip:** `dnf` replaced `yum`. Both are RPM-based. `zypper` is openSUSE only.

---

### dpkg-Based (Debian-Based) Distributions

| Distro | Notes |
|---|---|
| **Ubuntu** | Most popular Debian derivative — ease of use, large ecosystem |
| **Linux Mint** | User-friendly, Windows-like interface, based on Ubuntu |
| **Kali Linux** | Cybersecurity / penetration testing — specialized toolset |
| **Debian** | The upstream base — very stable, community-driven |

**Package format:** `.deb`

**Package managers:**
| Tool | Type | Behavior |
|---|---|---|
| `apt` | High-level | Auto-fetches packages AND dependencies from online repositories |
| `dpkg` | Low-level | Installs a pre-downloaded `.deb` file manually |

```bash
sudo apt install firefox           # Auto-download + install + handle deps
sudo dpkg -i firefox.deb          # Manual install of already-downloaded .deb
```

> **Exam tip:** `apt` handles dependencies automatically. `dpkg` does not — you must provide the `.deb` file yourself. The `-i` flag = install.

---

### Key Concepts: Package Management

- **Software dependency:** Libraries or components an application needs to run
- Package managers resolve dependencies automatically
- **Repositories:** Online collections of packages (Debian-based have extensive community repos)

---

### RPM vs dpkg Quick Reference

| | RPM-Based | dpkg-Based |
|---|---|---|
| Package format | `.rpm` | `.deb` |
| High-level tool | `dnf` / `yum` / `zypper` | `apt` |
| Low-level tool | `rpm` | `dpkg` |
| Common distros | RHEL, Fedora, openSUSE | Ubuntu, Debian, Kali |
| Target audience | Enterprise | Desktop + Security |
| `sudo` needed? | Yes | Yes |

---

## 7. Linux Graphical User Interface (GUI)

Unlike Windows or macOS, Linux lets you choose and swap out every layer of the GUI independently.

### The Four GUI Components

---

#### 1. X Server (X Window System / Xorg)

- The **original** Linux display system — in use since the 1980s
- Implements the **X11 display protocol**
- Acts as middleware between applications and the display
- Handles: window rendering, keyboard input, mouse input

**Unique feature — Network Transparency:**
- Run an app on one machine, display it on another over the network
- Implemented via **X forwarding over SSH**
- Example: Run LibreOffice on office workstation, see it on home laptop

**Drawbacks:**
- Outdated architecture → security vulnerabilities
- Performance inefficiencies
- Still widely used (Xorg) due to broad application compatibility

---

#### 2. Wayland

- Modern **replacement** for X Server
- Direct communication between application and display (fewer processing steps)
- Benefits: **better security, lower latency, smoother graphics**
- Security model: each app renders in its own isolated process — a crashing app cannot capture input from others
- Limitation: some tools (screen recorders, remote desktop apps) still require X Server
- Trend: gradually replacing X Server as software support improves

> **Exam tip:** Wayland = newer, more secure, more efficient. X Server = older, wider compatibility.

---

#### 3. Window Managers

Control how windows **move, resize, and behave** on screen.

| Type | Behavior | Example |
|---|---|---|
| **Floating** | Windows move/resize freely | Mutter (GNOME), KWin (KDE) |
| **Tiling** | Windows auto-arranged on a grid | i3, sway |

- **Mutter** = window manager for GNOME desktop
- **KWin** = window manager for KDE Plasma (most customizable)

---

#### 4. Display Managers

Provide the **graphical login screen** and launch the desktop session.

| Display Manager | Associated Desktop | Distro |
|---|---|---|
| **GDM** (GNOME Display Manager) | GNOME | Ubuntu |
| **SDDM** (Simple Desktop Display Manager) | KDE Plasma | Ubuntu KDE, Fedora KDE Spin |

> The display manager is what you see **before** you log in.

---

### GUI Layer Stack (top to bottom)

```
User interacts with apps
         ↓
   Desktop Environment (GNOME, KDE)
         ↓
   Window Manager (Mutter, KWin)
         ↓
   Display Manager (GDM, SDDM) — login screen
         ↓
   Display Protocol (X Server / Wayland)
         ↓
        Hardware
```

---

## 8. Software Licensing

### The Four License Types

---

#### Free Software

- Users can: **use, modify, share, and distribute** — no restrictions
- "Free" = **freedom**, not price
- Philosophical focus: **user rights**
- Associated organization: Free Software Foundation (FSF)

---

#### Open Source Software

- Source code is **publicly available**
- Focus: **collaboration, transparency, security**
- All Free Software is Open Source
- Not all Open Source is fully Free Software

**Example of the distinction:**
- **Linux** = both Free Software and Open Source
- **Google Chrome** = Open Source base (Chromium), but includes proprietary components → restricts some user freedoms

---

#### Proprietary Software

- Source code is **closed** — cannot view, modify, or distribute
- Vendor retains full control
- Users must purchase a license and agree to terms of use
- Examples: **Windows, macOS, Adobe Creative Suite**

---

#### Copyleft Software

A specific **licensing mechanism** used within FOSS.

**Core rule:** If you modify and distribute copyleft software, you **must** release your modifications under the same license.

- Primary license: **GNU GPL (General Public License)**
- Guarantees software remains free and open in all future versions
- Contrast: **permissive licenses** (e.g., Apache License) allow modified versions to become proprietary

> **Exam tip:** GPL = copyleft. Apache = permissive. Copyleft "infects" derivatives — they must stay open.

---

### License Comparison Table

| Type | Source Viewable | Modifiable | Redistributable | Key Restriction |
|---|---|---|---|---|
| Free Software | Yes | Yes | Yes | None — maximize user freedom |
| Open Source | Yes | Yes | Yes | Varies by specific license |
| Proprietary | No | No | No | Must buy license; vendor controls all |
| Copyleft (GPL) | Yes | Yes | Yes | Derivatives must use same license |

---

## Quick-Reference Cheat Sheet — Section 2

### Boot Process Commands
```bash
update-grub              # Regenerate GRUB2 config after editing /etc/default/grub
mkinitramfs              # Rebuild initrd (Debian/Ubuntu)
dracut                   # Rebuild initrd (RHEL/Fedora)
sysctl -p                # Apply /etc/sysctl.conf without reboot
sysctl net.ipv4.ip_forward=1    # Enable IP forwarding (runtime)
```

### Package Management Commands
```bash
# RPM-based
sudo dnf install <pkg>
sudo yum install <pkg>       # legacy
sudo zypper install <pkg>    # openSUSE only

# dpkg-based
sudo apt install <pkg>
sudo dpkg -i <file>.deb
```

### Key Files
| File | Purpose |
|---|---|
| `/boot/grub/grub.cfg` | GRUB2 boot config (auto-generated) |
| `/etc/default/grub` | GRUB2 admin settings (edit this) |
| `/boot/initrd.img` | Initial RAM disc image |
| `/etc/sysctl.conf` | Persistent kernel parameters |
| `/var/log/auth.log` | Authentication log (Debian-based) |

---

## Exam Gotchas — What They Like to Test

1. **PXE needs both DHCP and TFTP** — not just one of them
2. **Edit `/etc/default/grub`, then run `update-grub`** — never edit `grub.cfg` directly
3. **initrd loads simultaneously with the kernel** — not before, not after
4. **`mkinitramfs` = Debian/Ubuntu; `dracut` = RHEL/Fedora**
5. **`sysctl -p` applies `/etc/sysctl.conf` without rebooting**
6. **`apt` auto-resolves dependencies; `dpkg` does not**
7. **`dnf` replaced `yum`** but both appear on exams
8. **`-i` flag in `dpkg -i` means install**
9. **Wayland = newer, more secure; X Server = older, wider compat**
10. **Free Software ≠ free price — it means freedom**
11. **Copyleft (GPL) requires derivatives to stay open; Apache License does not**
12. **RISC-V = only open-source CPU architecture listed**
13. **x86 = 32-bit = 4 GB RAM hard limit**
14. **/tmp is cleared on reboot; /var is not**
15. **`/proc` and `/sys` are virtual — no actual files on disk**

---

## Section 2 Review Quiz — Practice Questions

**Q1.** What protocol does PXE use to get an IP address from the network?
> **A:** DHCP

**Q2.** What protocol does PXE use to download boot files?
> **A:** TFTP (Trivial File Transfer Protocol)

**Q3.** What command rebuilds the GRUB2 configuration file after editing `/etc/default/grub`?
> **A:** `update-grub`

**Q4.** What is the initrd and when is it loaded?
> **A:** A temporary root filesystem loaded simultaneously with the kernel, providing essential drivers before the real root filesystem mounts.

**Q5.** What command rebuilds the initrd on an Ubuntu system?
> **A:** `mkinitramfs`

**Q6.** What file stores persistent kernel parameters?
> **A:** `/etc/sysctl.conf`

**Q7.** What command applies `/etc/sysctl.conf` without rebooting?
> **A:** `sysctl -p`

**Q8.** Which directory holds personal user files?
> **A:** `/home`

**Q9.** What is the maximum RAM addressable by a 32-bit (x86) system?
> **A:** 4 GB

**Q10.** Which Linux architecture is fully open source and customizable?
> **A:** RISC-V

**Q11.** What package format do Debian-based distros use?
> **A:** `.deb`

**Q12.** What is the difference between `apt` and `dpkg`?
> **A:** `apt` automatically resolves and downloads dependencies from repositories. `dpkg` installs a pre-downloaded `.deb` file without handling dependencies.

**Q13.** What command would you use to install Firefox on Fedora?
> **A:** `sudo dnf install firefox`

**Q14.** What is Wayland's security advantage over X Server?
> **A:** Each application is isolated in its own rendering process, preventing a crashed or malicious app from capturing input from other apps.

**Q15.** What does a Copyleft license (GPL) require?
> **A:** Any modified version of the software that is distributed must be released under the same open-source license.

---

*Next: Section 3 — [paste content when ready]*

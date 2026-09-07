# Born2beRoot — A 42 Curriculum Study Guide

*Notes compiled and expanded by FALAMLIH*

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Virtualization & Hypervisors](#2-virtualization--hypervisors)
3. [Choosing a Distribution: Debian vs. Rocky (CentOS)](#3-choosing-a-distribution-debian-vs-rocky-centos)
4. [The Boot Process & GRUB](#4-the-boot-process--grub)
5. [Package Management: APT vs. Aptitude vs. DNF](#5-package-management-apt-vs-aptitude-vs-dnf)
6. [Filesystem & LVM](#6-filesystem--lvm)
7. [Networking & SSH](#7-networking--ssh)
8. [Mandatory Access Control: AppArmor vs. SELinux](#8-mandatory-access-control-apparmor-vs-selinux)
9. [Firewalls: UFW vs. firewalld](#9-firewalls-ufw-vs-firewalld)
10. [Users & Groups](#10-users--groups)
11. [Sudo Configuration & Hardening](#11-sudo-configuration--hardening)
12. [Password Policy (PAM)](#12-password-policy-pam)
13. [Scripting, Monitoring & System Tools](#13-scripting-monitoring--system-tools)
14. [Cron & Scheduled Tasks](#14-cron--scheduled-tasks)
15. [Key Configuration Files — Quick Reference](#15-key-configuration-files--quick-reference)
16. [Resources](#16-resources)

---

## 1. Project Overview

**Born2beRoot** is a project from the 42 curriculum that teaches students how to set up a **virtual machine** on a server from the ground up — partitioning, security hardening, permissions, package installation, and user/group management. It's essentially a hands-on introduction to Linux system administration: you build a small, hardened server and defend every configuration choice you made.

The core skills it exercises:
- Installing and partitioning a Linux system correctly (with LVM)
- Securing SSH access
- Enforcing strong password and `sudo` policies
- Configuring a firewall
- Writing a monitoring script
- Understanding *why* each security choice matters, not just *how* to apply it

---

## 2. Virtualization & Hypervisors

**Virtualization** is the process of running multiple operating systems on a single physical machine using a piece of software called a **hypervisor** (e.g. VirtualBox, UTM, QEMU).

### Two types of hypervisors

| Type | Description | Examples |
|---|---|---|
| **Bare metal** (Type 1) | Installed directly on the physical hardware, with no host OS underneath it | VMware ESXi, Proxmox, Xen |
| **Hosted** (Type 2) | Installed as an application on top of an existing operating system | VirtualBox, UTM, VMware Workstation |

### Why virtualize?
- Test multiple operating systems without touching or risking the host machine's main OS.
- Save **snapshots** — point-in-time backups of a VM's disk and state that you can roll back to instantly.
- Manage and cap the physical resources (CPU, RAM, disk) allocated to each guest independently.

### VirtualBox vs. UTM

| | **VirtualBox** | **UTM** |
|---|---|---|
| Vendor | Oracle | Developed for Apple platforms |
| Snapshots | Built into the UI — easy, user-friendly | Not exposed natively in the UI; you need the underlying **QEMU** tooling to snapshot |
| Best on | Intel/AMD (x86) machines | Apple Silicon (ARM) — takes advantage of native ARM virtualization |
| Guest OS support | Very broad Linux support, mature ecosystem | Strong on ARM-based guest OSes |

> Rule of thumb: on an Intel Mac or PC, **VirtualBox** is usually the smoother choice. On Apple Silicon, **UTM** performs better because it runs ARM guests natively instead of emulating x86.

To check whether a graphical session/interface is available on a machine:

```bash
ls /usr/bin/*session
```

---

## 3. Choosing a Distribution: Debian vs. Rocky (CentOS)

### Debian — general-purpose, maximum freedom

Debian is one of the **oldest and most stable** Linux distributions, and the base that Ubuntu (and many others) is built on. It uses **APT** as its package manager.

- Great for newcomers, desktops, and servers alike.
- Huge software repository.
- **SELinux is not enabled by default** (Debian favors AppArmor instead — see [§8](#8-mandatory-access-control-apparmor-vs-selinux)).
- Enormous amount of documentation and community support online.

| Pros | Cons |
|---|---|
| Stability & reliability | Packages tend to be older/more conservative versions |
| Vast software repositories | Initial setup can demand more technical knowledge |
| High flexibility, general-purpose | |

### Rocky Linux (RHEL family) — enterprise-grade

Rocky Linux is a downstream rebuild of **Red Hat Enterprise Linux (RHEL)**, aimed at businesses and legacy enterprise applications. It is not entirely "free" in the same permissive sense as Debian — it targets long-term, supported enterprise deployments. It uses **`dnf`** and **`rpm`** as package managers, which are powerful but somewhat less beginner-friendly than `apt`.

- **SELinux is enabled by default.**

| Pros | Cons |
|---|---|
| Enterprise focus, production-grade stability | Smaller software repository than Debian |
| Excellent for learning RHEL/CentOS-family administration | Less general-purpose, more specialized toward enterprise use |
| Desktop-friendly | |

Check your OS and kernel version:

```bash
uname -r              # kernel release version
cat /etc/os-release    # distro name, version, ID
```

---

## 4. The Boot Process & GRUB

**GRUB** (GRand Unified Bootloader) is responsible for loading the operating system's kernel into memory.

### The boot sequence, step by step

1. **Power on** — the firmware (legacy BIOS or modern UEFI) initializes the hardware.
2. The firmware hands control to the **bootloader** (GRUB).
3. **GRUB** takes over: it displays its boot menu (if configured with multiple entries), loads the chosen OS kernel into memory, and passes control to that kernel.
4. The kernel then initializes the rest of the OS (init system, services, etc.).

```
Power On → Firmware (BIOS/UEFI) → GRUB → Kernel → OS
```

GRUB stands out among bootloaders for its **versatility** and **feature-richness** — notably its ability to boot multiple different operating systems from the same machine (multi-boot).

---

## 5. Package Management: APT vs. Aptitude vs. DNF

Both **APT** and **Aptitude** are package-management tools for Debian-based systems; they operate on the same underlying `.deb` package format and repositories, just with different interfaces and levels of interactivity.

### APT
The **default** command-line tool for managing packages on Debian-based systems.
- Simple and fast — installs packages without prompting for extra options.
- Requires solid command-line/Linux knowledge, since everything happens through terminal commands with little hand-holding. This can be harder for a total beginner.

```bash
sudo apt update              # refresh the package index
sudo apt install <package>   # install a package
sudo apt upgrade             # upgrade all installed packages
sudo apt remove <package>    # remove a package
```

### Aptitude
A more **interactive** front-end that gives users richer options for managing packages and resolving dependency conflicts.
- Does **not** come pre-installed by default — you install it *with* `apt`:
  ```bash
  sudo apt install aptitude
  ```
- Its text-based interface abstracts away many of the sub-commands you'd otherwise need to remember for `apt` (install, upgrade, remove, etc.), making it friendlier for newcomers.

### DNF / RPM (Rocky/RHEL family)
The equivalent tooling on the Red Hat side of the world.

```bash
sudo dnf install <package>
sudo dnf update
sudo dnf remove <package>
```

To see the overall Linux filesystem hierarchy and what each top-level directory (`/etc`, `/var`, `/usr`...) is for:

```bash
man hier
```

---

## 6. Filesystem & LVM

**LVM (Logical Volume Manager)** is Linux's storage-management layer. It sits between your raw disks and the filesystems you actually use, giving you flexibility that plain partitions don't.

What LVM lets you do:
- **Create, resize, and manage disk partitions** *while the system is running*, without needing to unmount and repartition from scratch.
- **Group several physical disks** into a single logical volume — so your storage isn't locked to the size of one physical device.
- **Take snapshots** of volumes for backup purposes, without needing to shut the system down.

### LVM's three layers

```
Physical Volume (PV)  →  Volume Group (VG)  →  Logical Volume (LV)
   (raw disk/partition)     (pool of PVs)         (what you format & mount)
```

| Layer | What it is |
|---|---|
| **Physical Volume (PV)** | A raw disk or disk partition, initialized for use by LVM |
| **Volume Group (VG)** | A pool combining one or more PVs into a single storage pool |
| **Logical Volume (LV)** | A "virtual partition" carved out of a VG — this is what you actually format with a filesystem (ext4, xfs...) and mount |

List block devices and their mount points:

```bash
lsblk
```

### Mount points
A **mount point** is a directory in the filesystem where a partition or storage device gets attached, making its contents accessible as part of the overall directory tree (e.g. `/`, `/home`, `/var`).

> ⚠️ Note: a mount point is purely a *filesystem attachment point* — it has nothing to do with encryption or client-server connections. (That description belongs to SSH — see [§7](#7-networking--ssh).)

---

## 7. Networking & SSH

**SSH (Secure Shell)** is a cryptographic network protocol used to securely operate network services over an untrusted network like the internet. It provides an **encrypted, secure connection** between a client (your machine) and a server (the remote machine), protecting data — including credentials — from interception.

- SSH is essential for accessing virtual machines remotely: it lets you securely connect, execute commands, and manage applications/services without a physical console.
- It provides a **secure pseudoterminal**, giving the user direct, encrypted command-line communication with the server.

### IP addressing basics
An **IP address** is a unique identifier for every device connected to a network.
- An **IPv4** address is a 32-bit number, conventionally written as four decimal numbers (octets) separated by dots — each octet ranges from `0` to `255` (e.g. `192.168.1.42`).

### Common SSH commands

```bash
sudo service ssh status          # check whether the SSH service is running
ssh newuser@localhost -p 4241    # connect to a host on a custom port (4241 here)
```

Key SSH configuration files:

| File | Purpose |
|---|---|
| `/etc/ssh/sshd_config` | **Server-side** configuration (the SSH daemon — settings like port, root login, allowed users) |
| `/etc/ssh/ssh_config` | **Client-side** configuration (defaults for outgoing SSH connections you initiate) |

---

## 8. Mandatory Access Control: AppArmor vs. SELinux

Both **AppArmor** and **SELinux** are Linux kernel security modules that enforce **Mandatory Access Control (MAC)**.

> **MAC (Mandatory Access Control)** is a centralized access-control model where the *system* (not the resource owner) regulates access, based on fixed security policies — comparing the clearance level of a user/process against the security attributes of the object it's trying to access. This is stricter than the traditional discretionary permissions (`rwx`) model, where the file's owner decides who gets access.

### AppArmor
The **default** security module on Debian-based systems.

| Pros | Cons |
|---|---|
| Easy to learn and manage | Static profiles need manual updates whenever an app changes |
| Quick to set up | Not ideal for complex, highly dynamic environments |
| Uses human-readable path-based profiles | |
| A good fit for desktops and low-risk servers | |

### SELinux
The **default** security module on Red Hat Enterprise Linux, CentOS, Rocky Linux, and their derivatives.

| Pros | Cons |
|---|---|
| Enterprise-grade, very granular security | Complex to configure and manage |
| | Requires specialized tooling and know-how |
| | Steep learning curve for newcomers |

> **Key difference:** AppArmor confines programs based on **file paths**; SELinux confines them based on **security labels/contexts** attached to files and processes — which is more powerful but much more involved to administer.

---

## 9. Firewalls: UFW vs. firewalld

A **firewall** is network security software (or hardware) that monitors and controls incoming and outgoing traffic based on predefined rules — think of it as a gatekeeper deciding what data may enter or leave your machine or network.

> Only `root` (or a user with sufficient privileges) can manage the firewall — its binary typically lives at `/usr/sbin/ufw`.

### UFW (Uncomplicated Firewall)
A user-friendly front-end for managing `iptables` rules, and the default firewall tool on **Ubuntu/Debian-based** systems. It simplifies configuration with straightforward command-line syntax, aimed at users who don't want to wrestle with raw `iptables`.

```bash
sudo ufw status                # is it active?
sudo service ufw status        # check the service state
sudo ufw status numbered       # list current rules, numbered
sudo ufw allow 8080            # allow traffic on port 8080
sudo ufw delete <rule_number>  # remove a specific rule by its number
```

### firewalld
A high-level firewall management interface, default on many distributions including **RHEL/CentOS**. It supports **network zones**, predefined services, and "rich rules" for more nuanced policies — also designed to be user-friendly, but with a different rule model (zone-based) than UFW's simple allow/deny list.

### What firewalls are used for
- Securing web servers (only expose the ports you actually need, e.g. 80/443)
- Protecting SSH access (restrict to specific IPs, non-default ports)
- Managing which applications can accept incoming connections
- Blocking unwanted or malicious traffic

---

## 10. Users & Groups

### `su` — switch user

```bash
su
```
`su` allows commands to be run as a substitute user. Called with no username, it defaults to an interactive root shell. When a user is specified, extra arguments are passed straight to the new shell — **without** setting up that user's environment variables.

```bash
su -
```
`su -` (or `su -l`) also switches user, but loads the **full target environment** — default shell, environment variables, `$PATH`, home directory — as if that user had logged in directly. This is almost always what you actually want when fully "becoming" another user.

### `adduser` vs. `useradd`

| Command | Level | Behavior |
|---|---|---|
| `useradd` | Low-level | You must manually specify options like home directory, shell, etc. — nothing is created by default |
| `adduser` | High-level, user-friendly | Interactive; automatically creates a home directory, sets a default shell, prompts for a password, and more |

```bash
sudo adduser name_user             # create a new user (interactive, friendly)
sudo addgroup evaluating           # create a new group
sudo adduser name_user evaluating  # add an existing user to a group
```

### Changing the hostname

```bash
sudo nano /etc/hostname   # set the actual hostname
sudo nano /etc/hosts      # make sure 127.0.1.1 resolves to the same hostname
sudo reboot
hostname                  # verify the change after reboot
```

### Changing a password

```bash
passwd
```

### Checking users and groups

```bash
getent group sudo    # list members of the "sudo" group
getent passwd user42  # look up a specific user's account entry
groups user42          # list all groups a given user belongs to
id user42               # show UID, GID, and group memberships at once
```

---

## 11. Sudo Configuration & Hardening

Sudo policy is configured in a dedicated file under `/etc/sudoers.d/` (never edit `/etc/sudoers` directly — use `visudo` or a drop-in file, so a syntax error can't lock you out of `sudo` entirely):

```bash
sudo nano /etc/sudoers.d/sudo_config
```

### Common hardening directives

| Directive | Effect |
|---|---|
| `passwd_tries=3` | Limits the number of incorrect password attempts before failing |
| `badpass_message="..."` | Custom message shown on a wrong password |
| `logfile=`, `log_input`, `log_output` | Full auditing — logs sudo activity, including the commands typed and the output produced |
| `requiretty` | Only allows `sudo` to run from a **real terminal (TTY)** session |
| `secure_path=` | Restricts which directories/binaries `sudo` is allowed to execute from |

**Why `requiretty` matters:** it prevents a user's password from being leaked in cleartext through non-interactive channels — without a genuine TTY, background scripts, automated tools, or malicious services can't trick the system into silently running `sudo` commands on a user's behalf. In short: **`sudo` cannot be run by background scripts, cron jobs, or non-interactive sessions** when this is enforced.

### Checking sudo

```bash
sudo -l              # list what the current user is permitted to run with sudo
which sudo            # locate the sudo binary
```

### Sudo logs

```bash
cd /var/log/sudo
ls
cat sudo_config
```

---

## 12. Password Policy (PAM)

Password aging and complexity rules are enforced in two places.

### 1. Password aging — `/etc/login.defs`

```bash
sudo nano /etc/login.defs
```
Set the maximum and minimum number of days a password can remain valid (`PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE`).

### 2. Password complexity — `libpam-pwquality`

Install the password-quality PAM module:

```bash
sudo apt install libpam-pwquality
```

Then edit the PAM common-password rules:

```bash
sudo nano /etc/pam.d/common-password
```

| Setting | Meaning |
|---|---|
| `retry=3` | Number of retries allowed before the password prompt fails |
| `minlen=10` | Minimum number of characters a password must contain |
| `ucredit=-1` | Requires **at least one uppercase letter**. The `-` sign means "minimum required"; a `+` sign would instead cap a *maximum* |
| `dcredit=-1` | Requires **at least one digit** |
| `lcredit=-1` | Requires **at least one lowercase letter** |
| `maxrepeat=3` | Disallows the same character repeated **3 or more times in a row** |
| `reject_username` | The password cannot contain the account's username |
| `difok=7` | The new password must differ from the previous one by at least **7 characters** |
| `enforce_for_root` | Applies this entire password policy to the `root` account too, not just regular users |

---

## 13. Scripting, Monitoring & System Tools

A **script** is a sequence of commands stored in a single file, executed together to perform the combined function of each command.

| Tool | What it does |
|---|---|
| `grep` | Searches for specific text (words, phrases, or patterns) within files or input streams |
| `awk` | A scripting language for structured text processing — splits input into fields and lets you act on them |
| `free` | Command-line tool showing system memory (RAM & swap) usage: total, used, free, shared, buffer/cache |
| `vmstat` | Shows broad system statistics: process counts, memory usage, CPU activity, I/O, system status |
| `expr` | Evaluates expressions (arithmetic, string) from the command line and prints the result |
| `journalctl` | Views, filters, and manages system logs collected by the `systemd` journal |
| `wall` | "Write all" — broadcasts a message to every logged-in user's terminal |

### `print` vs. `printf` in `awk`

| | `print` | `printf` |
|---|---|---|
| Behavior | Limited; automatically appends a newline after its output | Modeled on C's `printf`; uses explicit **format specifiers** (`%s`, `%d`, ...) |
| Newline | Automatic | You must add `\n` yourself |
| Control | Less control over formatting | Precise control over spacing, padding, decimals, etc. |

```bash
awk '{print $1}' file.txt              # prints the first field of each line, auto-newline
awk '{printf "%-10s %d\n", $1, $2}' file.txt   # formatted, left-padded column output
```

### MAC address vs. IP address

| | **MAC Address** | **IP Address** |
|---|---|---|
| Layer | Physical (Data Link layer) | Logical (Network layer) |
| Identifies | The physical network interface hardware | A device's location on a network |
| Changes | Fixed to the hardware (though can be spoofed) | Can change depending on the network you're on |

---

## 14. Cron & Scheduled Tasks

**`crontab`** is a background process manager: the tasks you define run automatically at the times you specify, without needing to trigger them by hand.

```bash
sudo crontab -u root -e   # edit root's crontab file
```

Crontab syntax (five time fields, then the command):

```
# minute  hour  day-of-month  month  day-of-week   command
    0      2        *          *         *         /usr/local/bin/backup.sh
```

| Field | Range |
|---|---|
| Minute | 0–59 |
| Hour | 0–23 |
| Day of month | 1–31 |
| Month | 1–12 |
| Day of week | 0–7 (both 0 and 7 = Sunday) |

---

## 15. Key Configuration Files — Quick Reference

| File | Purpose |
|---|---|
| `/etc/ssh/sshd_config` | SSH server (daemon) configuration |
| `/etc/ssh/ssh_config` | SSH client default configuration |
| `/etc/sudoers.d/sudo_config` | Custom sudo policy / hardening rules |
| `/etc/login.defs` | System-wide login and password aging defaults |
| `/etc/pam.d/common-password` | PAM password-quality/complexity rules |
| `/etc/hostname` | The system's hostname |
| `/etc/hosts` | Static hostname-to-IP resolution |

---

## 16. Resources

- GRUB explained — [Lenovo Glossary: GRUB](https://www.lenovo.com/us/en/glossary/grub/)
- Linux filesystems compared (Ext2/Ext3/Ext4/XFS) — [Scribd document](https://fr.scribd.com/document/660125079/Linux-File-System-Ext2-vs-Ext3-vs-Ext4-vs-XFS)
- *The Linux Programming Interface* (reference book, PDF)
- Born2beRoot tracking/checklist tool — [born2beroot-tracking.pages.dev](https://born2beroot-tracking.pages.dev/)
- LVM overview — [Wikipedia: Logical Volume Manager (Linux)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))
- APT vs. Aptitude — [packagecloud.io blog](https://blog.packagecloud.io/know-the-difference-between-apt-and-aptitude/)
- Debian vs. Rocky Linux — [Medium article](https://amadla.medium.com/debian-linux-vs-rocky-os-exploring-the-best-choice-for-your-server-dfd6b3d80c1a)
- SELinux vs. AppArmor — [TuxCare blog](https://tuxcare.com/blog/selinux-vs-apparmor/)
- Mandatory Access Control explained — [NordLayer](https://nordlayer.com/learn/access-control/mandatory-access-control/)
- `su` vs. `su -` — [GeeksforGeeks](https://www.geeksforgeeks.org/linux-unix/difference-between-su-and-su-command-in-linux/)
- Linux firewall administration — [Pluralsight blog](https://www.pluralsight.com/resources/blog/software-development/linux-firewall-administration)

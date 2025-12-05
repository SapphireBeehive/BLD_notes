Here is a clean, comprehensive, professional **Markdown hardening guide** for BIOS/UEFI + GRUB lockdown to complement your YubiKey-secured Ubuntu system.

You can add this directly to your security documentation.

---

# 🔐 **Ubuntu Physical Security Hardening: BIOS/UEFI + GRUB Lockdown**

*(Prevents bypass of YubiKey login + YubiKey-protected sudo)*

This guide describes how to harden your workstation against “evil maid” attacks, unauthorized physical access, and root escalation via firmware or bootloader bypass.
It complements your existing configuration:

* **GUI login requires YubiKey only**
* **sudo requires YubiKey + password**

Without this hardening, an attacker with physical access could bypass all OS-level protections by manipulating boot settings or using GRUB recovery mode.

---

# 1. 🧩 Threat Model Overview

With physical access and no hardening, an attacker could:

* Boot your system from USB/DVD/network and overwrite your disk
* Enter BIOS/UEFI settings and disable Secure Boot
* Reset boot order
* Enter GRUB menu and:

  * Boot into *recovery mode* (→ root shell bypass)
  * Edit kernel parameters (`init=/bin/bash`)
  * Remove or bypass PAM/YubiKey enforcement
* Replace your kernel or initramfs
* Modify files on an unencrypted drive

**Hardware access = full compromise unless locked down.**

The steps below eliminate these attack paths.

---

# 2. 🔐 BIOS/UEFI Hardening Checklist

Enter BIOS/UEFI setup (F2, DEL, ESC, F10 depending on system).

## ✔ Set a **Supervisor (Admin) Password**

This prevents:

* Changing boot order
* Entering firmware setup
* Disabling Secure Boot

> **IMPORTANT:** Record this password securely.
> Without it, no one (including you) can change firmware settings.

---

## ✔ Optional: Set a **User Password**

This requires a password *before* the machine boots at all.

Recommended for laptops or high-risk environments.

Not required for desktops in secure locations.

---

## ✔ Disable Boot from External Devices

Set:

* **Boot from USB** → Disabled
* **Boot from CD/DVD** → Disabled
* **Boot from SD card** → Disabled
* **Boot from Network/PXE** → Disabled
* **Boot Menu Hotkeys (F12, ESC)** → Disabled (if available)

This prevents attackers from booting external tools.

---

## ✔ Enable **Secure Boot**

This prevents unsigned bootloaders and kernels from running.

Recommended settings:

* **Secure Boot → Enabled**
* **Secure Boot Mode → Standard** (or “Deployed Mode”)

This blocks:

* Bootkits
* Kernel replacement
* Non-signed GRUB edits

---

## ✔ Enable “Kernel DMA Protection” (if available)

Protects against DMA attacks through Thunderbolt.

---

## ✔ Disable Legacy Boot (CSM)

Use pure UEFI mode only.

---

# 3. 🔐 GRUB Bootloader Hardening

GRUB must be locked down so no one can:

* Drop into recovery → root shell
* Edit kernel boot parameters
* Boot older vulnerable kernels
* Boot with `init=/bin/bash` or disable PAM
* Mount the filesystem and remove your YubiKey requirements

### ✔ First: Generate a secure GRUB password

```bash
sudo grub-mkpasswd-pbkdf2
```

Enter a strong password. Output will look like:

```
grub.pbkdf2.sha512.10000.<long hash>
```

Copy the entire hash string.

---

## ✔ Add GRUB password to /etc/grub.d/40_custom

Edit:

```bash
sudo nano /etc/grub.d/40_custom
```

Add:

```bash
set superusers="root"
password_pbkdf2 root <your_pbkdf2_hash_here>
```

Save and exit.

---

## ✔ Protect recovery mode entries

GRUB uses “restricted” vs “unrestricted” mode:

* **Normal entries** → unrestricted (no GRUB password needed)
* **Recovery entries** → restricted (GRUB password required)

Edit `/etc/grub.d/10_linux` and ensure ONLY normal menuentries include:

```
--unrestricted
```

Recovery entries should **not** have `--unrestricted`.
This makes them require the GRUB password.

---

## ✔ Rebuild GRUB

```bash
sudo update-grub
```

---

# 4. 🔐 Full-Disk Encryption (Highly Recommended)

If not already using LUKS, set it up on next OS install.

Benefits:

* Prevents attackers from modifying `/etc/pam.d`
* Prevents copying `/etc/u2f_keys`
* Prevents extraction of user credentials
* Ensures physical theft = data theft impossible

Disk encryption + BIOS lockdown + GRUB password = **strong physical security posture**.

---

# 5. 🧯 Recovery Path Considerations

By design, after hardening:

* No one can enter GRUB recovery without password
* No one can boot external media
* No one can modify the OS without full-disk encryption password
* sudo still requires **YubiKey + password**
* GUI still requires **YubiKey-only login**
* TTY login remains password-only (intentional “break-glass” access)

Even in TTY login, without the GRUB or BIOS passwords,
attackers **cannot reach root** or bypass authentication.

---

# 6. 🔐 Summary: Final Security Posture

After completing these steps:

| Attack Type                | Protected? | How                                      |
| -------------------------- | ---------- | ---------------------------------------- |
| Physical access to device  | ✔️         | BIOS password + Secure Boot              |
| Booting external OS        | ✔️         | Disabled boot devices                    |
| Kernel parameter tampering | ✔️         | GRUB locked                              |
| GRUB recovery → root shell | ✔️         | GRUB password                            |
| Password-only login        | ✔️         | GUI login is YubiKey-only                |
| Password-only sudo         | ✔️         | sudo requires YubiKey + password         |
| Key theft (Bio or NFC)     | ✔️         | Backup key + password required           |
| Key loss                   | ✔️         | TTY break-glass login + recovery options |
| Drive removal / cloning    | ✔️         | With LUKS enabled                        |

With both BIOS + GRUB hardening applied, your system becomes:

### 🔥 Highly resistant to physical compromise

### 🔥 Immune to bootloader bypass

### 🔥 Secure against password theft

### 🔥 Enforcing hardware-backed login + sudo

### 🔥 Suitable for professional, corporate, or sensitive workloads

---

# 7. 📎 Optional: Add this Sticker Note to Your PC

> **WARNING:**
> BIOS, GRUB, and OS are hardened.
> Requires:
>
> * BIOS Supervisor Password
> * GRUB Password
> * YubiKey for GUI Login
> * YubiKey + Password for sudo
>
> Without proper credentials + YubiKey, access is impossible.

---

If you want, I can also generate:

* A PDF version
* A Bootloader Hardening Flowchart
* A script to automatically verify lock-down status
* A script to enforce Secure Boot + disable USB boot on supported devices

Just tell me!

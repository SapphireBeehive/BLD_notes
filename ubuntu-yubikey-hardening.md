# 🔐 Ubuntu Security Hardening: YubiKey + Password for Login & Sudo

This document summarizes the complete setup implemented on an Ubuntu system using **two hardware security keys**:

* **Primary:** YubiKey Bio (FIDO2 + on-device fingerprint)
* **Backup:** YubiKey 5 NFC (Full multi-protocol key)

The system now enforces **real 2-factor authentication** for both **login** and **sudo**:

> **You must have a registered YubiKey AND your account password.**
> No password-only or key-only fallback exists.

---

# 1. Overview of the Security Model

We configured Ubuntu to require **two factors**:

1. **Something you have** — a YubiKey (either the Bio or the 5 NFC)
2. **Something you know** — your system password
3. **Optional (Bio only)**: **Something you are** — fingerprint verification

This protects against:

* Password theft
* Remote attackers
* Keyloggers
* Unauthorized physical access
* Stolen devices (laptop/desktop)
* Loss/theft of one of the keys

Both keys can unlock sudo and login independently.

---

# 2. FIDO2 / pam_u2f Overview

Ubuntu integrates YubiKeys through the **pam_u2f** authentication module.

Key features of this module:

* Verifies the physical key by cryptographic challenge/response
* Requires **touch** (and fingerprint for Bio keys)
* Rejects non-present or spoofed keys
* Supports multiple keys per user
* Works without PIV or OpenPGP — ideal for FIDO-only Bio keys

We configured pam_u2f to require authentication via:

```
authfile=/etc/u2f_keys
```

This ensures login works even if `~/.config` isn’t mounted yet.

---

# 3. U2F Registration of Both Keys

Each key was registered using:

```bash
pamu2fcfg > ~/.config/Yubico/u2f_keys
pamu2fcfg -n >> ~/.config/Yubico/u2f_keys   # for the backup key
```

The registration file was then moved to a system-wide secure location:

```bash
sudo cp ~/.config/Yubico/u2f_keys /etc/u2f_keys
sudo chmod 600 /etc/u2f_keys
sudo chown root:root /etc/u2f_keys
```

### Final result

`/etc/u2f_keys` contains two entries:

* One for the primary YubiKey Bio
* One for the backup YubiKey 5 NFC

pam_u2f will accept **either key**.

---

# 4. Sudo: YubiKey + Password Required

We modified `/etc/pam.d/sudo` so sudo requires:

* YubiKey cryptographic presence
* Password authentication

### Key portion of `/etc/pam.d/sudo`:

```pam
auth required pam_u2f.so authfile=/etc/u2f_keys
auth required pam_unix.so
```

This enforces:

> **Touch the YubiKey → then enter password → sudo succeeds.**

Any missing step → sudo denied.

---

# 5. Login (GDM): YubiKey + Password Required

We modified **`/etc/pam.d/gdm-password`**, which controls graphical (GDM) login.

### Key changes:

```pam
auth    requisite       pam_nologin.so
auth    required        pam_succeed_if.so user != root quiet_success

# First factor: YubiKey (required)
auth    required        pam_u2f.so authfile=/etc/u2f_keys

# Second factor: password (required)
@include common-auth
```

This enforces:

* **YubiKey touch + fingerprint (Bio)** OR
* **YubiKey touch (5 NFC backup)**
* **AND system password**

Both factors must succeed for login.

---

# 6. Testing and Verification

### Login tests

* **Bio key only** → Login works with **fingerprint + touch + password**
* **Backup NFC key only** → Login works with **touch + password**
* **No key** → Login fails (correct)

### Sudo tests

* **Bio key** → YubiKey touch + fingerprint → password → sudo OK
* **Backup key** → touch → password → sudo OK
* **No key** → sudo denied

Everything behaves exactly as expected.

---

# 7. Benefits of the Setup

### ✔ True 2FA for both login and sudo

### ✔ No key-only or password-only fallback

### ✔ Bio key adds biometric verification

### ✔ Backup key prevents lockout

### ✔ Resistant to:

* Physical theft
* Password compromise
* Remote attacks
* Keyloggers
* Shoulder surfing
* File tampering
* USB key cloning

### ✔ No need for PIV or complex certificates

(using pam_u2f keeps this setup simple and reliable)

---

# 8. Optional Enhancements

You may extend this configuration with:

### 🔒 TTY 2FA Login

Mirror the same lines into `/etc/pam.d/login`.

### 🔑 FIDO2 SSH Keys

Use hardware-backed SSH keys via:

```bash
ssh-keygen -t ed25519-sk -f ~/.ssh/id_ed25519_sk
```

### 🔐 PIV Smartcard Mode (YubiKey 5 NFC only)

Use the 5 NFC key for certificate-based login.

### 🧯 Break-glass account

Create a non-admin local account with password-only login as emergency access.

---

# 9. Summary

You have implemented a hardened Ubuntu authentication system where:

* **Login requires YubiKey + password**
* **Sudo requires YubiKey + password**
* **Two independent keys are registered**
* **Either key can unlock the machine**
* **Bio key adds fingerprint verification**
* **Backup key ensures no lockout**

This is equivalent to enterprise-grade workstation security and is stronger than most default Linux configurations.

Here is a clean, step-by-step **checklist** (Markdown format) you can use anytime to re-implement **YubiKey + password 2FA for login + sudo** on a fresh Ubuntu system.

---

# ✅ **Ubuntu Hardening Checklist: YubiKey + Password for Login & Sudo**

This checklist assumes:

* You have **two FIDO2 keys** (primary + backup)
* You want:

  * **Sudo = YubiKey + password**
  * **Login = YubiKey + password**
  * **Either key** works
  * **No password-only fallback**

---

# 1. 📦 Install required packages

```bash
sudo apt update
sudo apt install libpam-u2f pamu2fcfg
```

---

# 2. 🔐 Register both keys via pam_u2f

### Create config directory:

```bash
mkdir -p ~/.config/Yubico
```

### **Register primary key:**

```bash
pamu2fcfg > ~/.config/Yubico/u2f_keys
```

### **Register backup key:**

```bash
pamu2fcfg -n >> ~/.config/Yubico/u2f_keys
```

### Lock down permissions:

```bash
chmod 600 ~/.config/Yubico/u2f_keys
```

---

# 3. 📁 Move key file to a safe system-wide location

```bash
sudo cp ~/.config/Yubico/u2f_keys /etc/u2f_keys
sudo chown root:root /etc/u2f_keys
sudo chmod 600 /etc/u2f_keys
```

This ensures login works even before your home directory mounts.

---

# 4. 🔒 Configure **sudo** to require YubiKey + password

Edit sudo PAM file:

```bash
sudo nano /etc/pam.d/sudo
```

Add these lines at the **top**:

```pam
auth required pam_u2f.so authfile=/etc/u2f_keys
auth required pam_unix.so
```

Save and test:

```bash
sudo -k
sudo whoami
```

Expect:

* Touch key → then password → success
* No key → fail

---

# 5. 🔐 Configure **GDM login** to require YubiKey + password

Edit:

```bash
sudo nano /etc/pam.d/gdm-password
```

Insert immediately after the first two “auth” lines:

```pam
# First factor: YubiKey
auth required pam_u2f.so authfile=/etc/u2f_keys

# Second factor: password
@include common-auth
```

Do not modify the session, SELinux, or account blocks below.

---

# 6. 🔐 (Optional but recommended) Secure **TTY login**

Edit:

```bash
sudo nano /etc/pam.d/login
```

Add at the top:

```pam
auth required pam_u2f.so authfile=/etc/u2f_keys
auth required pam_unix.so
```

---

# 7. 🧪 Test Login Flow

### Test primary key:

* Insert Bio → login requires touch + fingerprint → password
* Sudo requires touch → password

### Test backup key:

* Remove Bio, insert 5 NFC
* Login requires touch → password
* Sudo requires touch → password

### Test no key:

* Login → FAIL
* Sudo → FAIL

Correct behavior.

---

# 8. 🧯 Create an emergency “break glass” account (optional)

This is a **non-admin, password-only** local account, for emergencies only:

```bash
sudo adduser recovery
```

Do **not** grant sudo.

Store its password securely.

---

# 9. 🔁 Backup your `/etc/u2f_keys`

Save it offline or in a vault:

```
/etc/u2f_keys
```

You need this if you ever rebuild the system or migrate.

---

# 10. 🎉 Done — system is fully hardened

You now have:

* **Login = YubiKey + password**
* **Sudo = YubiKey + password**
* **2 independent hardware keys supported**
* **Password-only login/sudo disabled**
* **Biometric support on primary key**
* **Touch required on backup key**

This is enterprise-grade local authentication.


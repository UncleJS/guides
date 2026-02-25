

## Table of Contents

- [Beginner’s Guide to YubiKeys](#beginners-guide-to-yubikeys)
- [What is a YubiKey?](#what-is-a-yubikey)
- [Why use a hardware key instead of SMS / app codes?](#why-use-a-hardware-key-instead-of-sms-app-codes)
- [What does a YubiKey actually do?](#what-does-a-yubikey-actually-do)
- [Common YubiKey Types](#common-yubikey-types)
- [Key Concepts (Critical for Beginners)](#key-concepts-critical-for-beginners)
  - [1. The YubiKey does **not** store your password](#1-the-yubikey-does-not-store-your-password)
  - [2. Registering vs Using](#2-registering-vs-using)
  - [3. Always own a backup key](#3-always-own-a-backup-key)
- [Where can you use a YubiKey?](#where-can-you-use-a-yubikey)
- [How Authentication Works (Simplified)](#how-authentication-works-simplified)
- [First-Time Setup Strategy](#first-time-setup-strategy)
  - [Step 1 – Identify critical accounts](#step-1-identify-critical-accounts)
  - [Step 2 – Register keys](#step-2-register-keys)
  - [Step 3 – Test recovery paths](#step-3-test-recovery-paths)
- [Using YubiKeys on Linux (RHEL-Style)](#using-yubikeys-on-linux-rhel-style)
  - [Detecting the key](#detecting-the-key)
  - [Required packages](#required-packages)
  - [Checking key status](#checking-key-status)
  - [Common Pitfall (Linux)](#common-pitfall-linux)
- [OTP vs FIDO2 vs Passkeys (Beginner Confusion)](#otp-vs-fido2-vs-passkeys-beginner-confusion)
- [Practical Security Habits](#practical-security-habits)
- [Threats YubiKeys Help Mitigate](#threats-yubikeys-help-mitigate)
- [When a YubiKey is Overkill](#when-a-yubikey-is-overkill)
- [Loss & Disaster Recovery](#loss-disaster-recovery)
- [Frequently Overlooked Beginner Mistakes](#frequently-overlooked-beginner-mistakes)
- [Mental Model for Beginners](#mental-model-for-beginners)

---
## Beginner’s Guide to YubiKeys

*A practical, ground-up introduction to hardware security keys*

---


[↑ Goto TOC](#table-of-contents)

## What is a YubiKey?

Yubico’s **YubiKey** is a small hardware device that strengthens authentication by storing cryptographic secrets in tamper-resistant hardware. Instead of relying only on passwords, a YubiKey proves your identity using secure cryptography.

In simple terms:

* A password = something you **know**
* A YubiKey = something you **have**

Combining both dramatically reduces account takeover risk.

---


[↑ Goto TOC](#table-of-contents)

## Why use a hardware key instead of SMS / app codes?

Traditional 2FA methods (SMS, authenticator apps) improve security, but have weaknesses:

| Method                 | Weakness                       |
| ---------------------- | ------------------------------ |
| **SMS codes**          | SIM swap attacks, interception |
| **Authenticator apps** | Phishing, device compromise    |
| **Passwords only**     | Reuse, leaks, brute force      |

A YubiKey mitigates these by:

* Using **public-key cryptography** (no shared secrets)
* Resisting phishing (origin-bound authentication)
* Keeping secrets **non-extractable**

---


[↑ Goto TOC](#table-of-contents)

## What does a YubiKey actually do?

A YubiKey can perform multiple roles depending on protocol:

* **FIDO2 / WebAuthn** → Modern passwordless & phishing-resistant login
* **U2F** → Older FIDO standard (still widely supported)
* **OTP (One-Time Password)** → Legacy compatibility
* **Smart card (PIV)** → Certificates, enterprise auth
* **OpenPGP** → SSH / email / code signing keys
* **Challenge-Response** → Advanced custom flows

One physical key, multiple functions.

---


[↑ Goto TOC](#table-of-contents)

## Common YubiKey Types

![Image](https://cryptonest.co.uk/cdn/shop/products/Yubico-yubikey-5-nfc-close-up.png?v=1658426301\&width=1920)

![Image](https://m.media-amazon.com/images/I/41DkFsG8yEL._AC_UF894%2C1000_QL80_.jpg)

![Image](https://cdn.shopify.com/s/files/1/0576/5369/0523/products/YubiKey-5-Nano_4.jpg)

![Image](https://bert.org/assets/posts/yubikey/nano.jpg)

Typical variants:

| Model Type | Key Feature                   |
| ---------- | ----------------------------- |
| **USB-A**  | Older laptops / desktops      |
| **USB-C**  | Modern devices                |
| **NFC**    | Tap-to-auth on phones         |
| **Nano**   | Low-profile, stays plugged in |
| **Bio**    | Fingerprint verification      |

**Rule of thumb:** Choose connectors matching your devices.

---


[↑ Goto TOC](#table-of-contents)

## Key Concepts (Critical for Beginners)


[↑ Goto TOC](#table-of-contents)

### 1. The YubiKey does **not** store your password

It stores cryptographic keys. Your password still exists unless you move to passwordless login.

---


[↑ Goto TOC](#table-of-contents)

### 2. Registering vs Using

**Registering a key** = linking it to an account
**Using a key** = proving identity during login

Every service requires initial registration.

---


[↑ Goto TOC](#table-of-contents)

### 3. Always own a backup key

Hardware can be:

* Lost
* Damaged
* Left at home

Best practice:

* **Primary key** → Daily use
* **Backup key** → Stored safely

Failure to do this is the most common mistake.

---


[↑ Goto TOC](#table-of-contents)

## Where can you use a YubiKey?

Widely supported services include:

* Google accounts
* GitHub
* Microsoft accounts
* Password managers
* SSH / Linux login
* VPN systems
* Enterprise SSO platforms

Support usually appears as:

* “Security Key”
* “Passkey”
* “Hardware Token”
* “FIDO2”

---


[↑ Goto TOC](#table-of-contents)

## How Authentication Works (Simplified)

![Image](https://images.pingidentity.com/image/upload/f_auto%2Cq_auto%2Cw_auto%2Cc_scale/ping_dam/content/dam/picr/img/bl/2022/0909/Img-Blog-FIDO2-Passkeys-v02-1044x1490.png)

![Image](https://www.researchgate.net/publication/346351393/figure/fig5/AS%3A962179541000195%401606412887296/The-general-challenge-response-authentication-system.png)

![Image](https://www.w3.org/TR/webauthn-2/images/webauthn-registration-flow-01.svg)

![Image](https://images.ctfassets.net/23aumh6u8s0i/2xxX1F2zya0XdsiCRz5dXk/3c3350168175e59b2e41be8afda3e9fd/2-Registration.png)

High-level flow:

1. Website sends cryptographic challenge
2. YubiKey signs challenge internally
3. Site verifies signature using stored public key
4. Login succeeds

No shared secrets → phishing resistance.

---


[↑ Goto TOC](#table-of-contents)

## First-Time Setup Strategy


[↑ Goto TOC](#table-of-contents)

### Step 1 – Identify critical accounts

Prioritize:

* Email accounts (highest impact)
* Password manager
* Cloud providers
* Developer accounts (GitHub, etc.)

---


[↑ Goto TOC](#table-of-contents)

### Step 2 – Register keys

For each account:

1. Open **Security / 2FA settings**
2. Choose **Add Security Key**
3. Insert key or tap via NFC
4. Touch sensor when prompted
5. Name the key (e.g., `YubiKey-Primary`)

Repeat for backup key.

---


[↑ Goto TOC](#table-of-contents)

### Step 3 – Test recovery paths

Verify:

* Backup key works
* Recovery codes stored securely
* Account login from new device works

---


[↑ Goto TOC](#table-of-contents)

## Using YubiKeys on Linux (RHEL-Style)

YubiKeys integrate cleanly with Linux systems.

---


[↑ Goto TOC](#table-of-contents)

### Detecting the key

```bash
lsusb | grep -i yubico
```

Expected output resembles:

```
Bus XXX Device XXX: Yubico.com YubiKey
```

---


[↑ Goto TOC](#table-of-contents)

### Required packages

For smartcard / PAM / PIV workflows:

```bash
sudo dnf install yubikey-manager yubico-piv-tool pcsc-lite
sudo systemctl enable --now pcscd
```

---


[↑ Goto TOC](#table-of-contents)

### Checking key status

```bash
ykman info
```

Shows supported applications and firmware.

---


[↑ Goto TOC](#table-of-contents)

### Common Pitfall (Linux)

If the key appears non-functional:

* Ensure **pcscd** is running
* Check SELinux denials:

```bash
sudo ausearch -m avc -ts recent
```

Most FIDO2 usage works without extra drivers.

---


[↑ Goto TOC](#table-of-contents)

## OTP vs FIDO2 vs Passkeys (Beginner Confusion)

| Technology           | When Used                      |
| -------------------- | ------------------------------ |
| **OTP**              | Legacy systems                 |
| **U2F**              | Older web security key flows   |
| **FIDO2 / WebAuthn** | Modern standard                |
| **Passkeys**         | Passwordless UX built on FIDO2 |

Modern recommendation → **FIDO2 / Passkeys**

Avoid building new systems around OTP unless required.

---


[↑ Goto TOC](#table-of-contents)

## Practical Security Habits

Good practices:

* Register **two keys minimum**
* Store backup key separately
* Protect accounts with recovery codes
* Avoid sharing keys
* Keep firmware reasonably current

---


[↑ Goto TOC](#table-of-contents)

## Threats YubiKeys Help Mitigate

YubiKeys significantly reduce:

* Phishing attacks
* Credential stuffing
* Password database leaks
* Remote account takeover

They do **not** protect against:

* Malware on your device
* Logged-in session hijacking
* Poor account recovery policies

Hardware keys improve authentication, not overall device hygiene.

---


[↑ Goto TOC](#table-of-contents)

## When a YubiKey is Overkill

May be unnecessary for:

* Disposable accounts
* Low-risk throwaway services
* Systems without key support

Focus on high-impact accounts first.

---


[↑ Goto TOC](#table-of-contents)

## Loss & Disaster Recovery

If a key is lost:

1. Use backup key or recovery codes
2. Log into accounts
3. Remove lost key from security settings
4. Register replacement key

Treat keys like house keys.

---


[↑ Goto TOC](#table-of-contents)

## Frequently Overlooked Beginner Mistakes

❌ Buying wrong connector type
❌ Not registering backup key
❌ Forgetting recovery codes
❌ Confusing OTP with FIDO2
❌ Assuming key works everywhere

---


[↑ Goto TOC](#table-of-contents)

## Mental Model for Beginners

Think of a YubiKey as:

> **A cryptographic identity device, not a storage device**

It proves who you are; it does not hold your data.

---

[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)

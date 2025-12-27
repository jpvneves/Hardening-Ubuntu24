# Ubuntu Desktop - 24.04.3

---

## Overview
This hardening guide is specifically made for Ubuntu Desktop, taking into consideration that SSH won’t be used.\
Before installing Ubuntu or attempting any of these hardening measures, it is safest and strongly recommended to read through the guide and check and recheck every step. While most measures are safe, some, like “Remove Unnecessary Services”, are risky without proper study beforehand.\
While all sections are numbered these aren't in any particular order and could be applied at different points of the hardening process. For example: you should activate Secure Boot (UEFI) before installing Ubuntu and activate Disk Encryption (LUKS) while installing Ubuntu.\
This guide will be constantly improved, with new commands added or removed as needed.

---

## 1. Update the System
Keeping the system properly updated is the first thing anyone should do to ensure security.\
Every system has vulnerabilities, and outdated packages allow a malicious attacker to exploit those vulnerabilities to compromise the system. Keeping the system updated with the latest packages helps reduce risk and prevent exploitation.\
To ensure that you have the latest packages at all times, it is important to enable automatic security updates.

```bash
sudo apt update                            # Refreshes the package list from configured repositories.
sudo apt full-upgrade                      # Upgrades all installed packages to the newest versions.
sudo apt install unattended-upgrades       # Installs the tool that automatically installs security updates in the background.
sudo dpkg-reconfigure unattended-upgrades  # Launches a configuration dialog to enable or adjust automatic updates.
sudo apt autoremove                        # Removes packages that were installed as dependencies but are no longer needed.
sudo apt clean                             # Clears the local cache of downloaded package files.
```

---

## 2. Enable Firewall
Ubuntu comes with its own Uncomplicated Firewall (UFW).\
It controls network traffic to and from your computer, serving as a barrier between your system and the network.\
UFW simplifies firewall management by using straightforward terminal commands.
Enabling it reduces the attack surface by blocking unsolicited inbound traffic and protecting against network-based attacks.

```bash
sudo ufw default deny incoming   # Blocks all unsolicited inbound connections by default
sudo ufw default allow outgoing  # Allows normal desktop activity
sudo ufw enable                  # Activates firewall
sudo ufw status verbose          # Checks firewall activation
```

---

## 3. Enable AppArmor
Linux security relies on user permissions (root vs non-root). If an application is compromised, it has the same permissions as the user running it, meaning it can do everything that user can do.\
AppArmor is a security module that protects the system by restricting the permissions of individual programs. It uses profiles that define which files an application can read, write, or execute, which network resources it can access, and more. If an application attempts to perform an action outside of its assigned profile, AppArmor will block it and log the violation.\
Profiles are defined per application, not per user, and consist of rules that specify the allowed behavior of an application.\
- **Enforce mode** blocks and logs policy violations.
- **Complain mode** allows violations but logs them for review.
AppArmor is enabled by default on some Linux distributions, such as Ubuntu.

```bash
sudo aa-status                        # Check if it's running
sudo systemctl enable apparmor --now  # If it's not running activate it
sudo aa-enforce /etc/apparmor.d/*     # Switch profiles from complain to enforce if needed
```

---

## 4. Remove Unnecessary Services
Linux comes with many services enabled by default. Many of them are important and required for normal use.\
However, not all services are necessary, and the more services you have running, the larger your system’s attack surface becomes.\
It is therefore important to disable any service that you know you won’t need or use, or that does not have critical dependencies. If you disable an important service, the system might become unstable.\
If unsure, do not disable it.

```bash
systemctl list-unit-files --type=service | grep enabled  # List all the enabled services
systemctl cat (service name)                             # See what the service does
systemctl list-dependencies (service name)               # List all the dependencies of the service
sudo systemctl disable --now (service name)              # Disable the service
sudo apt purge (service name)                            # After a few days of use without the service, purge it if you want
```

---

## 5. Secure User Accounts
One of the largest attack surfaces of your system is the user accounts.\
If an attacker were able to access your user account, they would be able to do anything that user can do, including accessing all of the user’s files and even using root privileges via sudo.\
In order to secure your user accounts you need:
- Strong authentication
- Minimal user accounts
- Hardened sudo and root access
- Account lockout
- Session and screen protection

```bash
passwd                                                                                   # Change password to a stronger one
sudo passwd -l root                                                                      # Locks root account
sudo visudo                                                                              # Harden sudo
- Add: Defaults timestamp_timeout=(forget password minutes)                              # Forgets sudo password after some minutes
- Add: Defaults logfile="/var/log/sudo.log"                                              # Create a log file for sudo command usage
getent group sudo                                                                        # Ensure only trusted users belong to sudo group
sudo apt install libpam-tmpdir                                                           # Prevents users from accessing each other’s temporary files
sudo nano /etc/pam.d/common-auth                                                         # Edit PAM configuration
- Add: auth required pam_faillock.so preauth silent deny=(amount) unlock_time=(seconds)  # If account is already locked after 5 attempts lock for 15 minutes
- Add: auth [default=die] pam_faillock.so authfail deny=(amount) unlock_time=(seconds)   # Faillock on failure
- Add: auth sufficient pam_faillock.so authsucc deny=5 unlock_time=900                   # faillock on success
gsettings set org.gnome.desktop.session idle-delay (seconds)                             # How long the system can be idle before lockout  
gsettings set org.gnome.desktop.screensaver lock-enabled true                            # Force the screen to lock when the screensaver activates
```

---

## 6. Disk Encryption
Not all attacks are cyber attacks.\
Your computer might get stolen, so it's important to ensure that if that happens, your data is safely secured.\
Disk Encryption converts all the data on your disk to unreadable ciphertext. This ensures that only someone with the correct decryption key can decrypt the data.

It is recommended to encrypt the disk during the Ubuntu installation by using:
- **LUKS (Linux Unified Key Setup)**: It uses strong cryptography (AES-XTS) to encrypt your disk.

While it is possible to encrypt the disk post-installation it is generally not recommended.

---

## 7. Secure Boot
Some malware might be able to attack your system even before it boots.\
Secure Boot (UEFI) is a firmware security feature that only allows trusted cryptographically signed software to run during boot.\
This prevents boot-level malware from executing.\
UEFI is activated inside the firmware settings, so the options might differ from vendor to vendor.\
It is recommended to install Ubuntu in UEFI mode.


# Ubuntu Desktop - 24.04.3

---

## Overview
TODO

---

## Table of Contents
TODO

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
sudo ufw status                  # Verifies firewall rules
```

---

## 3. Enable AppArmor
Linux security relies on user permissions (root vs non-root). If an application is compromised, it has the same permissions as the user running it, meaning it can do everything that user can.\
AppArmor is a security module that protects the system by restricting the permissions of individual programs. It uses profiles that define which files an application can read, write, or execute, which network resources it can access, and more. If an application attempts to perform an action outside of its assigned profile, AppArmor will block it and log the violation.\
Profiles are defined per application, not per user, and consist of rules that specify the allowed behavior of an application.\
- **Enforce mode** blocks and logs policy violations.
- **Complain mode** allows violations but logs them for review.
AppArmor is enabled by default on some Linux distributions, such as Ubuntu.

```bash
sudo aa-status                        # Check if it's running
sudo systemctl enable apparmor --now  # If it's not running activate it
```

---

## 4. Remove Unnecessary Services
Linux comes with many services enabled by default. Many of them are important and required for normal use.\
However, not all services are necessary, and the more services you have running, the larger your system’s attack surface becomes.\
It is therefore important to disable any service that you know you won’t need or use, or that does not have critical dependencies.

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
If an attacker was able to get inside your user account they would be able to do anything that user can do as well as access all the user's files and use even root via sudo.\
In order to secure your user accounts you need:
- Strong authentication
- Minimal user accounts
- Hardened sudo and root access
- Account lockout
- Session and screen protection

```bash
passwd                                                        # Change password to a stronger one
sudo passwd -l root                                           # Locks root account
sudo apt install libpam-tmpdir                                # Prevents users from accessing other users tmp folder
sudo faillog -m (max failed attempts) -l (lockout seconds)    # Change parameters for failed login attempts
sudo visudo                                                   # Harden sudo
Add: Defaults timestamp_timeout=(forget password minutes)     # Forgets sudo password after some minutes
Add: Defaults logfile="/var/log/sudo.log"                     # Create log file of sudo command uses
```

---

## 6. Disk Encryption
Not all attacks are cyber attacks.\
Your computer might get stolen, so it's important to ensure that if that happens, your data is safely secured.\
Disk Encryption converts all the data on your disk to unreadable ciphertext. This makes it so that only someone that has the correct decryption key can get decrypt the data.

It is recommended to encrypt the disk during the Ubuntu installation by using:
- **LUKS (Linux Unified Key Setup)**: It uses strong cryptography (AES-XTS) to encrypt you disk.

While it is possible to encrypt the disk post-installation it is generaly not recommended.

---

## 7. Secure Boot
TODO

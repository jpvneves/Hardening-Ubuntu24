# Hardening - Ubuntu Desktop 24.04.3 (WORK IN PROGRESS)

---

## Overview
The aim of this project is to harden a Ubuntu Desktop 24.04.3 virtual machine, and to catalogue all implemented hardening measures in order to study their effects and the trade-offs they introduce to the system.\
The progression is going to be slow, as I add more and more hardening measures as I study and test them. They will only be added once I am confident about all their details.
Over time, I will also add automation and auditing tools to ensure compliance with benchmarks.

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
Linux comes with multiple different services enabled by default, many are important and needed for normal use.\
Not all of them are needed and the more you have the larger the attack surface of your system.\
It's important then, to disable any service you know you won't need or use or that doesn't have dependencies that are important.

```bash
systemctl list-unit-files --type=service | grep enabled  # List all the enabled services
systemctl cat (service name)                             # See what the service does
systemctl list-dependencies (service name)               # List all the dependencies of the service
sudo systemctl disable --now (service name)              # Disable the service
sudo apt purge (service name)                            # After a few days of use without the service, purge it if you want
```
List of services that can me safely disabled (always check):
```bash
avahi-daemon               # Local network discovery (mDNS / Bonjour, AirPlay, network printers)
avahi-daemon.socket        # Socket activation for avahi-daemon
cups                       # Printing system
cups-browsed               # Automatically discovers network printers
cups.path                  # Watches printer config paths
cups.socket                # Socket activation for CUPS
saned                      # Scanner daemon (SANE backend)
bluetooth                  # Bluetooth device support (headsets, keyboards, mice)
ModemManager               # Mobile broadband (3G/4G/5G USB modems)
whoopsie                   # Sends crash reports to Ubuntu
apport                     # Collects crash and error reports
gnome-remote-desktop       # GNOME screen sharing (RDP / VNC)
rpcbind                    # RPC port mapping (used by legacy NFS and some network services)
rpcbind.socket             # Socket activation for rpcbind
pcscd                      # Smart card reader support
NetworkManager-wait-online # Delays boot until network is fully online (mainly for servers)
```


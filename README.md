# Hardening - Ubuntu 24.04.3

---

## Overview
The aim of this project is to harden two Ubuntu virtual machines, one desktop and one server, and to catalogue all implemented hardening measures in order to study their effects and the trade-offs they introduce to the system.\
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

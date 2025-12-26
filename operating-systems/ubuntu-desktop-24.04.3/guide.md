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


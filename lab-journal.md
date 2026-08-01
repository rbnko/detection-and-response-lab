# Lab Journal

Running log of what I did, what broke, and how I fixed it.

## 2026-07-23
- Created GitHub repo
- Created isolated bridge 'vmbr1' for the VM's of the lab.

## 2026-07-24
- Created Ubuntu Server 24.04 VM for wazuh.

## 2026-07-29
- Deployed Wazuh (indexer + manager + dashboard) on 24.04 Ubuntu Server VM.
- Created static route to reach the Wazuh dashboard from desktop PC via the Proxmox host, bridging both networks.
- Created Debian 13.6 VM 'deb-target'.

## 2026-07-30
- Installed Wazuh agent on 'deb-target'.
- Security Configuration Assessment for 'deb-target' scores 42% before hardening (baseline reference).
- Created Debian 13.6 VM 'ansible'.

## 2026-07-31
- **FIX PASSWDLESS ANSIBLE USER AFTERWARDS (FOR NOW USE -K)**
- First hardening control: root SSH login fully disabled.

## 2026-07-31
- Added CIS-aligned controls: SSH timeouts (ClientAlive*), MaxAuthTries,
  empty-password block, UFW enabled-on-boot, fail2ban, telnet removal,
  clean MOTD, login banner (/etc/issue.net).
- SCA score moved from 42% to 43%.
**Takeaway:** Learned to use Ansible to remediate the configuration weaknesses
surface by Wazuh's SCA, closing the loop between detection and remediation.
Key concepts acquired: idempotency, configuration-as-code.

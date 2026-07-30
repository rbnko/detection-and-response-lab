# Lab Journal

Running log of what I did, what broke, and how I fixed it.

## 2026-07-23
- Created GitHub repo
- Created isolated bridge 'vmbr1' for the VM's of the lab.

## 2026-07-24
- Created 24.04 Ubuntu Server VM for wazuh.

## 2026-07-29
- Deployed Wazuh (indexer + manager + dashboard) on 24.04 Ubuntu Server VM.
- Created static route to reach the Wazuh dashboard from desktop PC via the Proxmox host, bridging both networks.
- Created 24.04 Ubuntu Server VM 'deb-target'.

## 2026-07-30
- Installed Wazuh agent on 'deb-target'.
- Security Configuration Assessment for 'deb-target' scores 42% before hardening (baseline reference).
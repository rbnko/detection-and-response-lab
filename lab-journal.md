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

## 2026-08-01
- Hardened the Debian target VM with help of Lynis.
- Audited the system with Lynis, checking the hardening index and the suggestions provided, mapping the most relevant ones and applying them.
- Hardening Index BEFORE / AFTER: 68% / 77%

## 2026-09-05
- Launched a SSH brute-force attack from a Kali machine against the hardened Debian target.
- Ran Hydra from Kali against the Debian machine with a small set of custom wordlists.
- fail2ban (which was applied during the hardening via Ansible playbook) detected the authentication attempts and banned the Kali machine IP.
- tcpdump on Debian captured 821 packets (check .pcap)
- From Wazuh Threat Intelligence, 38 authentication failures were registered.
- Key alert: rule.id 5551, level 10, "PAM: Multiple failed logins in a small period of time" (21:44:23).

## 2026-09-12
- Analized the tcpdump PCAP with Wireshark.
- SYN filtering: applied `tcp.flags.syn == 1 && tcp.flags.ack == 0 && ip.src == 10.10.10.30` to observe only the connection-initiating packets sent from Kali. Result: 141 attempts to open an SSH session.
- Isolated TCP stream: applied `tcp.stream == 0` to observe the first complete SSH connection in isolation. The client protocol identified as libssh instead of OpenSSH, which translates to non-human traffic. The connection only lasted 0.38 seconds since it's establishment to its closing which further more indicates non-human traffic.
- TCP Retransmissions: once fail2ban banned the Kali machine, the Debian machine firewall silently dropped its packets; Kali kept resending the SYNs.
- Since the packets were dropped instead of rejected, Kali kept retransmitting them.

## 2026-09-13 
- Correlated PCAP with Wazuh server logs: expanded a rule 5503 event and inspected the log, revealing the attemped username (root) and source IP (10.10.10.30). 
- The Wazuh log also MITTRE mapped the event with **T1110.001 / Password Guessing / Credential Access**.

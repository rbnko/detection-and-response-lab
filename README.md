# Detection & Response Lab
Blue-team home lab: harden a Linux server with Ansible, attack it from a Kali machine, and detect and stop the attack with fail2ban + Wazuh (SIEM).

## Architecture
Isolated Proxmox network (`detection-lab`), no home-LAN access (NAT only for updates). Four VMs:

- **ansible** — control node, runs the hardening playbook against the target.
- **deb-target** — hardened Debian 13 server, monitored, runs the Wazuh agent.
- **wazuh** — SIEM, collects logs and raises alerts.
- **kali** — attacker.

```mermaid
graph TB
    subgraph LAB["🔒 Isolated network — detection-lab — 10.10.10.0/24 (no route to home LAN)"]
        ANSIBLE["<b>ansible</b><br/>10.10.10.15<br/><i>Control node · IaC</i>"]
        KALI["<b>kali</b><br/>10.10.10.30<br/><i>Attacker · Hydra</i>"]
        TARGET["<b>deb-target</b><br/>10.10.10.20<br/><i>Debian 13 · hardened<br/>Wazuh agent + fail2ban</i>"]
        WAZUH["<b>wazuh</b><br/>10.10.10.10<br/><i>SIEM · logs + alerts</i>"]

        ANSIBLE -->|"hardens via SSH"| TARGET
        KALI -->|"SSH brute-force (T1110.001)"| TARGET
        TARGET -->|"agent forwards logs"| WAZUH
    end

    style TARGET fill:#1a3d1a,stroke:#3fb950,color:#fff
    style KALI fill:#3d1a1a,stroke:#f85149,color:#fff
    style WAZUH fill:#1a2a3d,stroke:#58a6ff,color:#fff
    style ANSIBLE fill:#3d331a,stroke:#d29922,color:#fff
    style LAB fill:#0d1117,stroke:#30363d,color:#8b949e
```

## Hardening (Ansible)
Target hardened with an Ansible playbook (repeatable, version-controlled):
- SSH: no root login, `MaxAuthTries 3`, no empty passwords, no X11/TCP/agent forwarding, verbose logging.
- Firewall (ufw): default-deny, only SSH + Wazuh ports open.
- Tooling: fail2ban, Lynis, rkhunter, debsums.
- Policies: password expiration, login banners, telnet removed.

**Lynis hardening index: 68 → 77.**

## Attack & Detection
- **Attack:** automated SSH brute-force with Hydra (MITRE ATT&CK T1110.001).
- **Prevention:** fail2ban detected the burst and banned the attacker in ~2s, before the valid password was reached.
- **Detection:** Wazuh fired rule 5503 (per-attempt) and correlation rule 5551 (level 10).
- **Forensics:** 821-packet PCAP in Wireshark — 40% TCP retransmissions (the ban), `libssh` banner + 0.38s connections (automated-traffic signature).

The hardening is what stopped the attack — defense and attack are two halves of the same story.

## Results
| Metric | Value |
|--------|-------|
| Lynis index | 68 → 77 |
| Technique | T1110.001 (Password Guessing) |
| Detection | Wazuh rules 5503 + 5551 |
| Response | fail2ban auto-ban (~2s) |
| Outcome | Blocked, no compromise |

## Structure
```
ansible/       Hardening playbook (Infrastructure as Code)
evidence/      pcaps · scans · screenshots
reports/       Incident report (PDF)
lab-journal.md Daily lab notes
```

## Docs
- Incident report: `reports/incident-report.pdf`
- Hardening playbook: `ansible/hardening_pb.yml`
- Lab journal: `lab-journal.md`

## Stack
Proxmox · Debian · Kali · Ansible · Wazuh · fail2ban · ufw · Lynis · Hydra · tcpdump · Wireshark

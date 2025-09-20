# HOMELAB — Kali & Metasploitable Lab README

**Purpose**
This document describes how I built and configured a small homelab for penetration testing and learning using Kali Linux and Metasploitable 2 in VirtualBox. It focuses on reproducible steps, secure handling of intentionally-vulnerable images, network isolation, and tips for expanding the environment into a full homelab.

---

## Table of Contents
1. Goals & Scope
2. Hardware & Host Requirements
3. Downloads & Files
4. Architecture overview
5. VirtualBox VM creation (Kali & Metasploitable)
6. Network configuration (isolation + internet access)
7. External storage for vulnerable VMs
8. Connectivity & validation
9. Promiscuous mode and packet capture
10. Post-setup exercises & expansion ideas
11. Backup, snapshotting & cleanup
12. Troubleshooting checklist
13. Security & operational notes
14. Useful commands & references

---

## 1. Goals & Scope
- Provide a safe, reproducible homelab to practice penetration testing techniques.
- Keep intentionally vulnerable systems (Metasploitable) isolated from the host and production networks.
- Allow attacker VM (Kali) internet access while retaining an isolated lab network between attacker and target.
- Make it portable and easy to store the vulnerable image offline when not in use.

This README is meant for a single-host VirtualBox-based homelab used for learning, testing, and demos.

---

## 2. Hardware & Host Requirements
- Host machine with VirtualBox installed (recommended: VirtualBox 6+).
- Minimum recommended RAM: 8 GB (more is better); allocate at least 2–4 GB to Kali.
- Disk space: allow 20–40 GB for Kali (depending on tools) and ~5–10 GB for Metasploitable (or as the VM requires).
- Optional: external USB drive or NAS for storing vulnerable images offline.

---

## 3. Downloads & Files
- **Kali Linux ISO** — official Kali download page (choose the image for your architecture).
- **Metasploitable 2** — intentionally vulnerable VM image (usually provided as a VM folder or OVA).
- Keep checksums or source links in a `downloads` folder for reproducibility.

---

## 4. Architecture overview
- **Kali VM (attacker)**
  - Adapter 1: Bridged (or NAT) — allows internet access for updates and external tools.
  - Adapter 2: Host-only — isolates lab traffic and allows communication with targets.
- **Metasploitable VM (target)**
  - Adapter 1: Host-only — only reachable from the host and other VMs on the host-only network.
- **Host-only network (vboxnet0)**
  - Provides an isolated private network for attacker-target interactions.

This setup prevents the vulnerable VM from being exposed to your home/office network while letting Kali connect to the internet as needed.

---

## 5. VirtualBox VM creation (Kali & Metasploitable)
- Kali:
  1. Create a new VM (Linux → Debian 64-bit for Kali x64).
  2. Attach the downloaded Kali ISO to the virtual optical drive and perform a normal installation.
  3. Configure two NICs (see Section 6).
- Metasploitable:
  1. Import the OVA or add the VM folder via `Machine → Add...`.
  2. Ensure it is configured with a single Host-only adapter (vboxnet0).

---

## 6. Network configuration (isolation + internet access)
**Kali**
- Adapter 1: Bridged Adapter (or NAT if preferred). Use this for OS updates and downloading tools.
- Adapter 2: Host-Only Adapter → `vboxnet0` (for lab communication).

**Metasploitable**
- Adapter 1: Host-Only Adapter → `vboxnet0` (isolated target).

**Rationale**
- Bridged (or NAT) lets Kali reach external resources while host-only isolates vulnerable systems.
- All lab traffic should occur over the host-only network so it never leaves the host machine.

---

## 7. External storage for vulnerable VMs
Two recommended options:
- **Move VM folder to external drive**
  - Power off VM, move the VM folder containing `.vbox` and disk files to the external drive, then `Machine → Add...` to register the VM from that location.
- **Export as OVA**
  - Use `File → Export Appliance` to create an OVA you can store securely and import later.

Keep the external drive disconnected when not in use if you want physical separation.

---

## 8. Connectivity & validation
From Kali, confirm host-only IP and check reachability:

```bash
# On Kali
ip a           # find host-only interface and IP (e.g., 192.168.56.x)
ping -c 4 <METASP_IP>

# On Metasploitable
ifconfig
ping -c 4 <KALI_HOSTONLY_IP>
```

If pings succeed, the host-only network is operating correctly.

---

## 9. Promiscuous mode and packet capture
- I set promiscuous mode through the **VirtualBox GUI** (no CLI required):
  1. In VirtualBox Manager, right-click the VM → **Settings** → **Network**.
  2. Select the adapter (e.g., Adapter 2 for Kali's host-only interface).
  3. Change **Promiscuous Mode** to **Allow All** for Kali when you want to capture traffic.
  4. For Metasploitable, keep Promiscuous Mode at **Deny** or **Allow VMs** to limit exposure.

- If you prefer the command line, `VBoxManage` can also set promiscuous mode; these examples are optional.

`VBoxManage` examples (optional):

```bash
VBoxManage modifyvm "KaliVM" --nicpromisc2 allow-all
VBoxManage modifyvm "Metasploitable" --nicpromisc1 deny
```

Capture traffic on Kali:
```bash
sudo tcpdump -i <hostonly-interface> -w lab_capture.pcap
# or use Wireshark for GUI analysis
```

---

## 10. Post-setup exercises & expansion ideas
- Run common scans: `nmap -sS -sV -A <METASP_IP>`.
- Practice exploiting known Metasploitable services with Metasploit.
- Add more VMs: Windows vulnerable image, an internal DNS server, or an ELK stack for logs.
- Automate snapshots and daily backups to practice lab recovery.

---

## 11. Backup, snapshotting & cleanup
- Use VirtualBox snapshots before major changes.
- Export appliances (OVA) for long-term backups.
- When finished with a session: power off vulnerable VM, disconnect external drives, and delete temporary snapshots if necessary.

---

## 12. Troubleshooting checklist
- **No IP on host-only:** Verify network settings in VM and reboot the VM.
- **Kali has no internet:** Confirm Adapter 1 is bridged/NAT and the host NIC is up.
- **Moved VM won't start:** Check file permissions and `.vbox` paths; re-register the `.vbox` file.

---

## 13. Security & operational notes
- Treat Metasploitable as permanently vulnerable — never connect it to untrusted networks.
- Use host-only networks to keep lab traffic contained on the host.
- Consider running the homelab inside an air-gapped machine or dedicated host when doing high-risk experiments.

---

## 14. Useful commands & references
- `VBoxManage list hostonlyifs`
- `VBoxManage modifyvm "<vm>" --nicpromisc2 allow-all`
- `ip a` / `ifconfig`
- `nmap`, `tcpdump`, `wireshark`, `metasploit`

---


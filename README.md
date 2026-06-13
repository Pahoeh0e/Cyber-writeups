# SOC Home Lab
I got locked out of my own firewall three times building this. Here's what I learned.
A fully functional Security Operations Center (SOC) environment built from scratch on consumer hardware. This lab simulates enterprise network segmentation, perimeter security, and attack/defend workflows used by real security teams.

# Architecture Overview
| Layer                   | Technology                  | Purpose                                   |
| ----------------------- | --------------------------- | ----------------------------------------- |
| Hypervisor              | Proxmox VE 8.x              | Virtualization, RBAC, resource management |
| Firewall/Router         | pfSense CE                  | Perimeter security, NAT, DHCP, VPN        |
| Network Segmentation    | Linux Bridges (vmbr0/vmbr1) | WAN/LAN isolation                         |
| Attack Platform         | Kali Linux                  | Red team exercises, traffic generation    |
| SIEM (Week 3)           | Wazuh + Elastic Stack       | Log aggregation, threat detection         |
| Windows Domain (Week 2) | Windows Server + Windows 10 | Endpoint telemetry, Active Directory      |
| Automation (Week 4)     | Shuffle SOAR                | Incident response playbooks               |

# Network Topology


    Internet
      |
      
    Home Router (192.168.1.1/24)

            |
    +-- Proxmox Host (192.168.1.x)
            |
          
            +-- vmbr0 (WAN) --+-- pfSense WAN (192.168.1.x, DHCP)
            
            |                          |
           
            |                  +-- pfSense LAN (10.0.0.1/24)
           
            |                          |
            +-- vmbr1 (LAN) <----------+
          
                    |
           
                    +-- Kali Linux (10.0.0.100/24)
            
                    +-- Windows Server (10.0.0.101/24) [Week 2]
          
                    +-- Windows 10 (10.0.0.102/24) [Week 2]
              
                    +-- Wazuh SIEM (10.0.0.50/24) [Week 3]

                    
# Week 1: Foundation & Network Segmentation

### Proxmox Setup:

- Role-Based Access Control (RBAC): Created socadmin (administrator) and analyst1 (read-only) accounts with group-based permission management
- Authentication: Implemented TOTP two-factor authentication for privileged access
- Resource Management: Optimized CPU overcommitment across VMs on 2-core hardware

### pfSense Firewall Configuration:

- WAN Interface: DHCP from home router, double-NAT lab setup
- LAN Interface: Static 10.0.0.1/24 with DHCP server (range: 10.0.0.100-200)
- Security Policy: Default-deny WAN policy with explicit HTTPS allow rule for management access
- Key Lesson: The "Block private networks" rule blocked my own laptop because WAN is RFC1918 behind a home router. Fixed by adding a specific source IP allow rule.

### Kali Linux Deployment:

- Installation: Fresh install from official ISO (text mode for reliability)
- Network: VirtIO adapter on vmbr1 (isolated SOC LAN)
- Tools Installed: nmap, masscan, hydra, gobuster, enum4linux, bloodhound.py
- Key Lesson: A corrupted .qcow2 conversion from VMware taught me the difference between boot disks and data disks. Fresh ISO install was the cleanest fix.

# Troubleshooting Log:

- ## Problem: Can't Access pfSense Web UI from Laptop
Root Cause: Default-deny WAN policy + "Block private networks" rule treated my laptop (192.168.1.x) as a threat.
Fix: Added explicit firewall rule allowing HTTPS from my laptop's specific IP. Learned that lab WAN interfaces are often RFC1918, not public IPs.

- ## Problem: Kali VM Boot Loop — "No Bootable Device"
Root Cause: Corrupted .qcow2 disk image — likely converted a VMware snapshot instead of the base disk.
Fix: Destroyed broken VM, performed fresh install from ISO. Learned to verify image integrity with qemu-img info and fdisk -l before import.

- ## Problem: GRUB Rescue Prompt — normal.mod Not Found
Root Cause: Incomplete root filesystem on imported disk. Missing /etc, /boot, /dev, /var, /home directories.
Fix: Confirmed disk was unrecoverable via chroot and fsck. Reinstalled from scratch.


# What I Learned:

- Default deny is real. I locked myself out twice before I was able to change the firewall rule order by temporarily taking down the implicit block.
- Troubleshooting beats memorization. When GRUB broke, I had to understand what a bootloader actually does — not just copy commands.
- Documentation matters. If I can't explain it, I don't understand it. This README is proof of comprehension.
- Network topology first. Understanding double-NAT vs. direct public IP changes how you apply every security rule.

# Tools & Technologies:
Virtualization: Proxmox VE, QEMU/KVM, Linux bridges
Firewall: pfSense CE, stateful inspection, NAT, DHCP, OpenVPN
Endpoint: Kali Linux, nmap, masscan, hydra, gobuster, enum4linux
Identity: RBAC, group-based permissions, TOTP 2FA
Networking: VLAN segmentation, double-NAT, DHCP relay, DNS forwarding


## License
This project is for educational and portfolio purposes. All third-party software (Proxmox, pfSense, Kali Linux) is subject to their respective licenses.

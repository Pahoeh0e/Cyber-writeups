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

                    

## License
This project is for educational and portfolio purposes. All third-party software (Proxmox, pfSense, Kali Linux) is subject to their respective licenses.

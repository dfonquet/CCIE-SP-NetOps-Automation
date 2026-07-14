# NetLab-Core

## Hostname

```
netlab-core
```

---

## Purpose

Central Automation Platform for:

- CCIE Service Provider Lab
- Cisco 350-901 AUTOCOR
- Python Network Automation
- Ansible
- pyATS / Genie
- REST APIs
- NETCONF / RESTCONF
- Docker
- Terraform
- Cisco NSO
- FreeRADIUS
- TACACS+
- Future Monitoring Stack (Grafana / Prometheus)

---

## Operating System

- Ubuntu Server 26.04 LTS
- Linux Kernel 7.0.0-27-generic

---

## Virtualization

- Platform: EVE-NG Community
- Hypervisor: QEMU / KVM
- Architecture: x86_64

---

## Network Design

### WAN Interface

| Parameter | Value |
|-----------|-------|
| Interface | wan0 |
| Address | DHCP |
| Current IP | 192.168.1.99/24 |
| Gateway | 192.168.1.1 |
| Purpose | Internet Access, GitHub, apt, Docker Hub |

---

### Management Interface

| Parameter | Value |
|-----------|-------|
| Interface | mgmt0 |
| Address | 10.255.23.10/24 |
| Gateway | None |
| Network | NetOps-MGMT |
| Purpose | Device Management, SSH, NETCONF, RESTCONF, Automation |

---

## Connectivity Validation

Validated:

- Internet connectivity
- DNS resolution
- SSH access
- Default routing
- Management network connectivity

Current status:

```
ping 1.1.1.1       OK
ping google.com    OK
SSH                OK
```

---

## Installed Software

### Development

- Git
- Python 3.14
- pip
- Python Virtual Environment (venv)

### Automation Frameworks

- Ansible Core
- ansible-lint
- pyATS
- Genie
- Netmiko
- Scrapli
- NAPALM
- ncclient
- Paramiko
- Jinja2
- Requests
- PyYAML
- AsyncSSH
- GitPython
- Cisco ISE SDK
- Juniper PyEZ

### Utilities

- curl
- jq
- tcpdump
- tmux
- htop
- tree

---

## Repository

```
~/netops/CCIE-SP-NetOps-Automation
```

Repository structure:

```
automation/
docs/
full-configs/
README.md
```

---

## Current Resources

| Resource | Value |
|----------|------:|
| vCPU | 2 |
| RAM | 4096 MB |
| Disk | 19 GB |
| Swap | 2 GB |

---

## Future Components

Pending installation:

- Docker Engine
- Docker Compose
- Terraform
- NetBox
- Grafana
- Prometheus
- FreeRADIUS
- TACACS+
- Cisco NSO

---

## Design Notes

NetLab-Core is the central automation platform of the Service Provider laboratory.

The server separates Internet connectivity from the management plane:

```
Internet
    │
   wan0
    │
NetLab-Core
    │
  mgmt0
    │
NetOps-MGMT
```

This design allows all automation traffic (SSH, NETCONF, RESTCONF, pyATS and Ansible) to remain isolated from Internet access while maintaining connectivity for software updates and source code repositories.


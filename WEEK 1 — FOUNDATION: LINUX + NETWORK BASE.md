# Build the 30-day system engineering mission.


## WEEK 1 — FOUNDATION: LINUX + NETWORK BASE

## Day 1 — Lab Setup (Your “Data Center”)

### Tasks

**Install virtualization (VirtualBox / VMware)**
**Create 3 VMs:**

-   lb-1
-   app-1
-   db-1

**Commands**

```hostnamectl set-hostname lb-1```

Set static IP (Ubuntu example):

```sudo nano /etc/netplan/01-netcfg.yaml```
```
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.100.101/24
      routes:
        - to: default
          via: 192.168.100.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```
```sudo netplan apply```

**Trap**

-   Wrong gateway → no internet  
    👉 Debug with:

```ip a  ```
```ip route  ```
```ping 8.8.8.8```

----------

**Day 2 — SSH Hardening**

```sudo nano /etc/ssh/sshd_config```

Change:

```PermitRootLogin no  ```
```PasswordAuthentication no```

Restart:

```sudo systemctl restart ssh```

**Trap**

Lock yourself out  
👉 Fix via VM console

----------

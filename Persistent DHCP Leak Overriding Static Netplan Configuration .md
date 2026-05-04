

# 🧾 Incident Report & Fix Guide 
 
## Persistent DHCP Leak Overriding Static Netplan Configuration  
  
  
## 📍 Environment  
  
- **OS:** Ubuntu Server (Netplan + systemd-networkd)  
- **Interface:** `ens33`  
- **Platform:** VMware VM  
- **Target Config:**  
- IP: `192.168.100.101/24`  
- Gateway: `192.168.100.1`  
  
## 🚨 Problem Summary  
  
Despite configuring a static IP via Netplan, the system kept receiving a **DHCP IP (`192.168.1.x`)**, resulting in:  
  
- Dual IP addresses on a single interface  
- Conflicting gateways  
- Routing instability  
- Continuous DHCP lease loops  
  
  
## 🔍 Root Cause  
  
The issue was **NOT inside Linux**.  
  
> 🔥 **VMware Virtual Network DHCP server was overriding OS-level configuration**  
  
### Key Insight:  
- Even with `dhcp4: no`, the VM still receives DHCP offers  
- `systemd-networkd` accepts those offers unless explicitly blocked  
- Final authority = **Network Layer (VMware DHCP)**  
  
  
## 🛠️ FINAL FIX (Step-by-Step)  
  

  
# 🔴 PHASE 1 — Disable DHCP in VMware  
  
### 1. Open Virtual Network Editor  
- VMware → `Edit` → `Virtual Network Editor`  
- Click **Change Settings (Admin)**  
    
  
### 2. Identify active network  
  
Check VM settings:  
  
| Mode  | Network |  
|----------|--------|  
| NAT  | VMnet8 |  
| Bridged | VMnet0 |  
| Host-only| VMnet1 |  
  
  
### 3. Disable DHCP  
  
Select your VMnet → Uncheck:

[ ] Use local DHCP service to distribute IP address

  
  
### 4. Apply changes

Apply → OK

  
### 5. (Recommended)  
  
Switch VM to:

Host-only (VMnet1)

- Subnet: `192.168.100.0/24`  
- DHCP: Disabled  
  
  
# 🔴 PHASE 2 — Clean Linux State  
  
### 6. Flush interface  
  
```bash  
sudo ip addr flush dev ens33  
sudo ip route flush dev ens33
```


### 7. Remove DHCP leases

```sudo rm -rf /run/systemd/netif/leases/*```


### 8. Reset interface

```
sudo ip link set ens33 down  
sudo ip link set ens33 up
```

----------

## 🔴PHASE 3 — Enforce Static Config

### 9. Create systemd-networkd config

```sudo nano /etc/systemd/network/10-ens33.network```


```
[Match]  
Name=ens33  
  
[Network]  
DHCP=no  
Address=192.168.100.101/24  
Gateway=192.168.100.1  
DNS=8.8.8.8  
DNS=1.1.1.1
```



### 10. Neutralize Netplan

```sudo nano /etc/netplan/01-network-manager-all.yaml```

    network:
      version: 2
      renderer: networkd
      ethernets:
        ens33: {}



### 11. Regenerate config

```sudo netplan generate```

----------

### 12. Restart networking

```sudo systemctl restart systemd-networkd```


## **🔴** **PHASE 4 — Verification**

### 13. Check IP

```ip a```

✅ Expected:

```192.168.100.101```

❌ Should NOT exist:

```192.168.1.x```


### 14. Check routes

```ip route```

✅ Expected:

```default via 192.168.100.1```


### 15. Check network status

networkctl status ens33

✅ Expected:

DHCP: no


### 🧪 Final Stability Test

```watch -n 2 ip a```

✔ Wait 2–3 minutes
✔ No DHCP IP should reappear

----------

## **🚨** **Troubleshooting Checklist**

If issue persists:

```ls /etc/netplan/```

✔ Only ONE config file should exist

----------

**Verify VMware again:**

-   DHCP disabled ✔
-   Correct VMnet ✔
-   No NAT/Bridged conflict ✔

----------

**🧠 Key Learnings**

**1. OS is NOT the final authority**

Network infrastructure (VMware) can override OS config


**2. DHCP is external behavior**

Even static systems can receive DHCP unless blocked


**3. systemd-networkd is active, not passive**

It will continuously attempt DHCP unless disabled


**4. Virtualization adds hidden layers**

VMware acts as:

-   DHCP server
-   virtual switch
-   network policy engine

----------

**🧭 Final Architecture**

VMware Network (DHCP Disabled)  
↓  
systemd-networkd (Static Only)  
↓  
Linux Kernel (Single IP, Stable Route)

----------

**🚀** **Outcome**

You now have:

-   Deterministic static IP
-   No DHCP interference
-   Clean routing table
-   Production-grade network behavior

----------

**📌** **Author**

Miftah

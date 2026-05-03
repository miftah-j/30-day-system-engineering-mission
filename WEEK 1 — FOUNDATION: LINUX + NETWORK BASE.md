# Build the 30-day system engineering mission.


## WEEK 1 — FOUNDATION: LINUX + NETWORK BASE

## Day 1 — Lab Setup (Your “Data Center”)

### Tasks

Create Your Lab (DO NOT RUSH)

**Goal**

3 machines that can talk to each other.

**Action**

Install virtualization (VirtualBox / VMware)

Create 3 VMs:

-   lb-1 → 192.168.100.101
-   app-1 → 192.168.100.102
-   db-1 → 192.168.100.103

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


Test:
ping 192.168.56.11
If ping fails → STOP and fix.


**Trap**

-   Wrong gateway → no internet  

👉 Debug with:

```ip a  ```

```ip route  ```

```ping 8.8.8.8```


----------

## DAY 2 — Access Control (SSH)

**On your laptop:**

    ssh-keygen -t rsa  
    ssh-copy-id user@192.168.56.11

**Disable password login:**

```sudo nano /etc/ssh/sshd_config```

Set:

> PermitRootLogin no   
> PasswordAuthentication no

Restart:

```sudo systemctl restart ssh```

**Test:**

> Open NEW terminal → SSH again

**Trap**

If login fails → you locked yourself out  
👉 Use VM console to fix

----------

## DAY 3 — Firewall Thinking (VERY IMPORTANT)

On app-1:

    sudo apt install ufw -y  
    sudo ufw default deny incoming  
    sudo ufw allow ssh  
    sudo ufw enable

Test:

-   SSH works
-   Other ports blocked

**Break It:**

```sudo ufw deny ssh```

**Fix It (via console):**

```sudo ufw allow ssh```

----------

## Day 4 — Networking Between VMs

Test:

```ping 192.168.56.11```

**Break It**

```sudo ufw deny from 192.168.56.0/24```

**Fix**

```sudo ufw delete deny from 192.168.56.0/24```

----------

## Day 5 — DNS Setup (Local Resolution)

    sudo apt install dnsmasq -y  
    sudo nano /etc/dnsmasq.conf

Add:

```address=/app.local/192.168.56.11```

Restart:

```sudo systemctl restart dnsmasq```

**Trap**

DNS not resolving  
👉 Debug:

```systemctl status dnsmasq  ```
```dig app.local```

----------

## Day 6 — Git Setup (Professional Layer)

``` git init infra-lab ```
``` cd infra-lab ```

Structure:

> /infra   
> /docs   
> /scripts

Commit:

    git add .  
    git commit -m "Initial infra setup"

----------

## Day 7 — Review + Break Day

Break:

-   Stop networking
-   Change IP incorrectly
-   Disable SSH

Fix everything.

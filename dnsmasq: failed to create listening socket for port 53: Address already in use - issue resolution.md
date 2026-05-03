

# $ dnsmasq: failed to create listening socket for port 53: Address already in use - issue resolution

**🔴** **Root cause**

> dnsmasq: failed to create listening socket for port 53: Address already in use

**DNS Setup (Local Resolution)**

    sudo apt install dnsmasq -y
    sudo nano /etc/dnsmasq.conf
    Add: address=/app.lan/192.168.100.102

You’re on the right track—but there’s a hidden architectural gap that will break this setup if you don’t address it.

You added:
`address=/app.lan/192.168.100.102`

That part is correct **for static DNS mapping**.

But right now, dnsmasq isn’t even running (you already saw the port 53 conflict). So this config is **inactive**.

----------

**🔴** **Pressure-test your setup**

Before touching config, ask:

“Who is actually answering DNS queries on this machine right now?”

Because unless dnsmasq owns port 53, this line does nothing.

**🔍** **Step 1 — Verify DNS ownership**

Run:

`sudo ss -tulnp | grep :53`

If you see:

`127.0.0.53:53`

👉 That’s **systemd-resolved**, not dnsmasq.

**⚠️** **Core problem**

You now have:
-   dnsmasq (installed but failed)
-   systemd-resolved (active and already serving DNS)

👉 So your custom domain app.local will **never resolve**.

**✅** **Correct setup (clean and intentional)**

If your goal is a **real local DNS lab**, do this properly.

**Step 2 — Disable systemd-resolved**

    sudo systemctl stop systemd-resolved  
    sudo systemctl disable systemd-resolved

Fix resolver:

    sudo rm /etc/resolv.conf  
    echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

**Step 3 — Configure dnsmasq properly**

Edit:

`sudo nano /etc/dnsmasq.conf`

Add (don’t just add one line—this is where most people go shallow):

 **Bind only to your interface (adjust if needed)**  
 
    interface=ens33  
    bind-interfaces  
  
**Local DNS mapping**  

`address=/app.lan/192.168.100.102`

**Step 4 — Start dnsmasq**

`sudo systemctl restart dnsmasq `
`sudo systemctl status dnsmasq`

**Step 5 — Point system to your DNS**

`echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf`

**Step 6 — Test like an engineer**

`ping app.lan`

OR better:

`dig app.lan @127.0.0.1`

Expected:

> 192.168.100.102

Good — dnsmasq is running. But don’t celebrate yet, because your setup still has a **design flaw**.

----------

**🧠 First-principles view**

A DNS stack should be:

Either:

-   dnsmasq = resolver + forwarder  
    **OR**
-   systemd-resolved = resolver

Not both stacked blindly.

**🔍** **Now the critical question**

Before fixing anything:

👉 **Does** **app.local** **resolve?**

Test:

`dig app.lan`

If it fails, then:

dnsmasq is running but **not being used by your system**

**🚨** **Most likely issue**

Your system is still pointing to:

> 127.0.0.53

👉 That means:

-   systemd-resolved is still the primary resolver
-   dnsmasq is being bypassed for local queries

**✅** **Fix (clean architecture)**

You need to decide

We recommend make dnsmasq the primary DNS

###  Make dnsmasq the primary DNS (recommended for your lab)

**Step 1 — Point system to dnsmasq**

``sudo rm /etc/resolv.conf ``
``echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf``

**Step 2 — Prevent systemd-resolved from interfering**

Either fully disable:

```
sudo systemctl disable systemd-resolved  
sudo systemctl stop systemd-resolved
```

**Step 3 — Configure upstream DNS in dnsmasq**

Edit:

`sudo nano /etc/dnsmasq.conf`

Add:

```
server=8.8.8.8  
server=1.1.1.1
```

👉 Now dnsmasq is your **only DNS brain**

**🧪 Validate properly**

Run:

`dig app.lan @127.0.0.1`

Then:

`ping app.lan`

**🧠 Strategic insight**

Right now you’re learning something most people miss:

DNS problems are rarely DNS problems — they are **architecture problems**

**✅** **Best practice (what professionals do)**

Open the file:

`sudo nano /etc/dnsmasq.conf`

**🔍** **Step 1 — Find a clean section**

Search for existing relevant directives:

Ctrl + W → type: interface

-   If you find existing interface= lines → **modify them**
-   If not → create a clean custom section

**✍️** **Recommended placement**

Scroll to the **bottom of the file** and add a clearly separated block:

    === CUSTOM LAB CONFIG ===  
      
    # Bind only to your interface  
    interface=ens33  
    bind-interfaces  
      
    # Local DNS mapping  
    address=/app.lan/192.168.100.102

**⚠️** **But here’s the real trap (don’t skip this)**

**1. Interface name might be WRONG**

Run:
`ip a`

You might see:
-   ens33
-   eth0
-   ens160

👉 If ens33 doesn’t exist → dnsmasq will silently misbehave.

**2. You might not even need** **interface=**

If you're just doing **local DNS (same machine)**:

👉 Simpler and safer [If unsure] :

    listen-address=127.0.0.1  
    listen-address=192.168.100.102

**🔄** **After editing (this is mandatory)**

`sudo systemctl restart dnsmasq`

Then verify:

`sudo systemctl status dnsmasq`

**🧪 Test properly (not casually)**

`dig app.lan @127.0.0.1`

Then:

`ping app.lan`

**🚀** **My push (don’t stay basic)**

Instead of one record, do this:

```
address=/lb.lan/192.168.100.101
address=/app.lan/192.168.100.102  
address=/db.lan/192.168.100.103   
```

Then simulate:
-   App → DB via hostname
-   Load balancer → app nodes

**✅** **Exact placement (practical answer)**

Open:

sudo nano /etc/dnsmasq.conf

Scroll to the **bottom** and create a clean, explicit block:

    # === CUSTOM DNS CONFIG ===  
      
    # Upstream DNS servers  
    server=8.8.8.8  
    server=1.1.1.1  
      
    # Local DNS mapping  
    address=/app.lan/192.168.100.102  
      
    # Bind settings (adjust interface if needed)  
    interface=ens33  
    bind-interfaces

👉 Bottom of the file is fine **as long as you’re not duplicating earlier directives**.

**🔴** **But here’s the part most people miss**

dnsmasq already reads:

`/etc/resolv.conf`

Your output earlier showed:

> using nameserver 127.0.0.53

👉 That means:

-   It’s still forwarding to **systemd-resolved**
-   Your server=8.8.8.8 lines will **override that**, but only if properly applied

**⚠️** **Critical rule (this is where configs break)**

If you have BOTH:

> server=8.8.8.8
> 
> AND systemd-resolved active → you get **mixed resolution paths**

👉 Not wrong, but messy and harder to debug.

**🧠 Clean architecture (what you should aim for)**

Pick ONE:

**Path A (recommended for your lab)**

-   dnsmasq = **primary DNS**
-   systemd-resolved = **disabled**

**Path B (less ideal)**

-   dnsmasq = cache layer
-   systemd-resolved = upstream

**🔄** **After adding the lines**

``sudo systemctl restart dnsmasq``

**🧪 Validate properly (don’t skip)**

Check upstream servers being used:

``nslookup google.com 127.0.0.1``

OR:

``dig google.com @127.0.0.1``

**🔍** **Deep verification (pro move)**

`sudo tcpdump -i any port 53`

👉 You should see queries going to:

    -   8.8.8.8
    -   1.1.1.1

NOT `127.0.0.53`

**⚠️** **Common failure patterns**

If it doesn’t work, it’s usually:

1.  ❌ Duplicate server= lines elsewhere
2.  ❌ Wrong interface binding (ens33 mismatch)
3.  ❌  /etc/resolv.conf still pointing wrong
4.  ❌ dnsmasq not actually being used by system

**🧠 Strategic takeaway**

This is not about adding two lines.

It’s about controlling:

**Resolution path: Client → Resolver → Upstream**

----------

**🧪 Now test properly (not just blindly)**

**1. Test your local DNS (dnsmasq)**

    dig app.lan @127.0.0.1

Expected:

> -Returns 192.168.100.102

**2. Test external resolution via dnsmasq**

`dig google.com @127.0.0.1`

👉 Confirms:

> -dnsmasq is forwarding to upstream (8.8.8.8 / 1.1.1.1)

**3. Check what your system is actually using**

`cat /etc/resolv.conf`

If you see:

`nameserver 127.0.0.1`

👉 Good — system is using dnsmasq

If you see:

`nameserver 127.0.0.53`

👉 Still using systemd-resolved (your dnsmasq config is being bypassed)

**⚠️** **Fast workaround (no install)**

If you want to test _right now_ without installing anything:

`ping app.lan`

Not perfect, but gives a quick signal.

**🚀** **Next-level move**

Trace DNS flow:

`dig google.com @127.0.0.1 +trace`

👉 This shows **full resolution path**, not just the result.


----------
----------


## **More Troubleshooting:**

> user@appserver1:~$ cat /etc/resolv.conf
> nameserver 8.8.8.8
> user@appserver1:~$
> user@appserver1:~$ ping app.local
> ping: app.local: Name or service not known

Now we’ve got a **real diagnostic signal**, not noise.

You currently have:

> /etc/resolv.conf → 8.8.8.8

and:

> ping app.lan → Name or service not known

👉 That tells us something very specific:

Your system is bypassing dnsmasq completely.

**🔴** **Root cause (precise)**

You configured:

-   dnsmasq with address=/app.lan/...
-   dnsmasq is running
-   BUT your system DNS is: 8.8.8.8

**Why this breaks everything:**

Google DNS:

-   does NOT know app.lan
-   does NOT forward unknown .lan domains to your dnsmasq

So resolution flow is:

> ping → 8.8.8.8 → FAIL (no knowledge of app.lan)

dnsmasq is **not even being consulted**

----------

**🧠 First-principles truth**

DNS only works if:

Client → points to resolver that knows the answer

Right now:

-   Your resolver = Google DNS ❌
-   Your custom domain exists only in dnsmasq ❌
-   So they are disconnected

----------
----------

## **🔧** **Final Fix (clean and correct) with Upgraded config suggestion**

You must make **dnsmasq your local resolver**

**✅** **Step 1 — Point system to dnsmasq**

Run:

`sudo nano /etc/resolv.conf`

Change to:

`nameserver 127.0.0.1`

**⚠️** **IMPORTANT (Ubuntu will overwrite this)**

> Ubuntu often resets /etc/resolv.conf via systemd.

So do the proper fix:

    sudo systemctl disable systemd-resolved  
    sudo systemctl stop systemd-resolved

Then lock resolv.conf:

    sudo rm /etc/resolv.conf  
    echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf

**🔍** **Step 2 — Verify dnsmasq is actually answering**

`dig app.lan @127.0.0.1`

If this works but ping doesn’t → system resolver issue  
If both fail → dnsmasq config issue

**✅** **What is working correctly**

This line proves dnsmasq is functioning:

> SERVER: 127.0.0.1#53

And this is your success case:

> app.lan → 192.168.100.102

👉 Meaning:

-   dnsmasq is running ✔
-   local DNS mapping is active ✔
-   query is resolving correctly ✔

So the DNS layer is **not your problem anymore**.

----------

**🔴** **3. Missing local-only control (important for labs)**

Right now dnsmasq may still behave like a **general resolver + forwarder**, which is not what you want in a controlled lab.

**Add this for clarity:**

    domain-needed  
    bogus-priv

**✅** **What you did right**

> address=/lb.lan/192.168.100.101   
> address=/app.lan/192.168.100.102  
> address=/db.lan/192.168.100.103

This gives you:

-   Centralized DNS ✔
-   Human-readable service names ✔
-   Decoupling from IPs ✔

👉 That’s exactly how real environments work.

**⚠️** **Hidden risks (don’t ignore these)**

**1. Single point of failure**

Right now:

All servers → ONE dnsmasq → resolution

If dnsmasq dies → **everything breaks**

👉 In production, you’d never run only one DNS server.

**2. No redundancy / failover**

Your config is **static A records**:

-   If app.lan server goes down → DNS still resolves → traffic fails

👉 DNS ≠ health-aware by default

**3. No load distribution**

Right now:

lb.lan → single IP

But real systems often do:

app.lan → multiple IPs

## **🚀** **Upgrade your config (high-value improvement)**

**🔹** **Add multiple backends (DNS-based load balancing)**

    address=/app1.lan/192.168.100.102  
    address=/app2.lan/192.168.100.104

👉 dnsmasq will return both IPs (round-robin behavior)

**🔹** **Add TTL control (important for testing failover)**

    local-ttl=30

👉 prevents long DNS caching

**🔹** **Add logging (for debugging like a pro)**

    log-queries  
    log-facility=/var/log/dnsmasq.log

**🧪 How to validate properly**

Run multiple times:

`dig app.lan @127.0.0.1`

👉 You should see:

-   Same IP (single setup) OR
-   Multiple IPs (if you added more)

**⚠️** **One more thing (important)**

Make sure ALL servers use your DNS:

`cat /etc/resolv.conf`

Should point to:

> nameserver 192.168.100.102

If not → your whole design is bypassed.


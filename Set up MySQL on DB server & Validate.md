
**🧱 STEP 1 — Install MySQL on DB Server (192.168.100.103)**

    sudo apt update  
    sudo apt install mysql-server -y

Enable service:

    sudo systemctl enable mysql  
    sudo systemctl start mysql

Check status:

    sudo systemctl status mysql

**🔐** **STEP 2 — Secure MySQL (important even in lab)**

    sudo mysql_secure_installation

Recommended answers:

> -   VALIDATE PASSWORD: optional (lab → can skip strict)
> -   Remove anonymous users: ✔ yes
> -   Disallow root remote login: ✔ yes
> -   Remove test DB: ✔ yes
> -   Reload privileges: ✔ yes

**🌐** **STEP 3 — Enable remote access (CRITICAL STEP)**

Edit MySQL config:

    sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

Find:

> bind-address = 127.0.0.1

Change to:

> bind-address = 0.0.0.0

Restart MySQL:

    sudo systemctl restart mysql

**👤** **STEP 4 — Create test database user**

Login:

    sudo mysql

Run:

    CREATE DATABASE labdb;  
      
    CREATE USER 'labuser'@'192.168.100.%' IDENTIFIED BY 'Lab@1234';  
      
    GRANT ALL PRIVILEGES ON labdb.* TO 'labuser'@'192.168.100.%';  
      
    FLUSH PRIVILEGES;  
    EXIT;

**🔥** **STEP 5 — Firewall rules (VERY IMPORTANT)**

Allow only app/loadbalancer access:

    sudo ufw allow from 192.168.100.101 to any port 3306 proto tcp  
    sudo ufw allow from 192.168.100.102 to any port 3306 proto tcp  
    sudo ufw reload

**🧠 STEP 6 — DNS validation (db.lan)**

On DNS server (dnsmasq host):

    sudo nano /etc/dnsmasq.conf

Add:

> address=/db.lan/192.168.100.103

Restart:

    sudo systemctl restart dnsmasq

**🧪 STEP 7 — Test DNS + network**

From app server or LB:

    ping db.lan

Expected:

> 64 bytes from 192.168.100.103

**🧪 STEP 8 — Test MySQL connectivity (REAL VALIDATION)**

Install client on app server:

    sudo apt install mysql-client -y

Connect:

    mysql -h db.lan -u labuser -p

Enter password:

> Lab@1234

**🎯** **Expected Result**

You should see:

Welcome to the MySQL monitor

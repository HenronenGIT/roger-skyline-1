<h1 align="center">roger-skyline-1 </h1>

<!-- <h2><em><p align="center" style="italic">System and Network administration</p></em></h2> -->

<!-- > <h3 align="center">"System and Network administration"</h3> -->

<h2 align="center">
    <a href="#-description-">Description</a>
    <span>·</span>
    <a href="#-usage-">Usage</a>
    <span>·</span>
    <a href="#-testing-">Testing</a>
</h2>

# Virtual Machine

## Disk partitions

> 1.  You have to run a Virtual Machine with the Linux OS of your choice in the hypervisor of your choice.
> 2.  Disk size has to be 8GB
> 3.  At least one 4.2 GB partition.

My choice was to use Debian as a OS and Virtual Box as a hypervisor.\
Partitions for the Virtual Machine is easiest to do when you are installing VM for the first time. Partition can be done afterwards also, but it can be little bit cubersome. In this project when we knew exact size for partition beforehand it can be done at installation phase.

How to check partitions from the terminal:\
`lsblk`

![Partitions](./README/img/partitions.png)

## Packages

Updating packages

`apt-get update`\
`apt-get upgrade`

# Network and Security

> Create a non-root user to connect to the VM.

    useradd <name>

> Make this user

    `apt install sudo`

    `sudo vim /etc/sudoers`

```
hmaronen  ALL=(ALL) NOPASSWD:ALL
```

1.  We don’t want you to use the DHCP service of your machine. You’ve got to
    configure it to have a static IP and a Netmask in \30.
    `sudo vim /etc/network/interfaces`

![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%201.png)

[Network of VirtualBox instances with static IP addresses and Internet access. - Codes And Notes](https://www.codesandnotes.be/2018/10/16/network-of-virtualbox-instances-with-static-ip-addresses-and-internet-access/)

1.  You have to change the default port of the SSH service by the one of your choice.
2.  SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT
    be allowed directly, but with a user who can be root.
    `sudo vim /etc/ssh/sshd_config`

```c
// /etc/ssh/sshd_config
Port 2021

PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
```

`sudo systemctl restart sshd`

-   Please note that port numbers 0-1023 are reserved for various system services

[](https://www.cyberciti.biz/faq/howto-change-ssh-port-on-linux-or-unix-server/)

[How to add SSH public key to server](https://www.simplified.guide/ssh/copy-public-key)

---

1.  You have to set the rules of your firewall on your server only with the services used
    outside the VM.
    `sudo apt install ufw`

        `sudo ufw default deny incoming`
        `sudo ufw default allow outgoing`

```c

// HTTP
sudo ufw allow 80/tcp
// HTTPS
sudo ufw allow 443
//SSH
sudo ufw allow 2021/tcp

// Enabled on startup
sudo vim /etc/ufw/ufw.conf
	ENABLED=yes
//Start
sudo ufw enable
//Check
sudo ufw verbose
```

---

1. You have to set a DOS (Denial Of Service Attack) protection on your open ports
   of your VM.

## fail2ban

-   **apache2 need to be installed to get right path in fail2ban config file.**
-   Create copy from jail.conf → jail.local
    -   `cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
    -   jail.conf may be overwritten by update
        `sudo fail2ban-client status <name of jail>` - check ip:s banned

```bash
sudo apt-get install iptables apache2 fail2ban

// Main configure fails
**/etc/fail2ban/fail2ban.conf**
/etc/fail2ban/jail.conf

// Filter file what needs to be created
/etc/fail2ban/filter.d/http-get-dos.conf

// patterns to filter out what is normal server activity
ignoreregex

// Show active jails of fail2ban
sudo fail2ban-client status

//banned IP:s
sudo tail -f /var/log/fail2ban.log
//or
sudo fail2ban-client status sshd
```

## Testing with Slowloris

`sudofail2ban-client status | grep "Jail list:" | sed "s/ //g" | awk '{split($2,a,",");for(i in a) system("fail2ban-client status " a[i])}' | grep "Status\|IP list"`

[Slowloris DDOS Attack Tool in Kali Linux - GeeksforGeeks](https://www.geeksforgeeks.org/slowloris-ddos-attack-tool-in-kali-linux/)

[Fail2Ban Port 80 to protect sites from DOS Attacks | TO THE NEW Blog](https://www.tothenew.com/blog/fail2ban-port-80-to-protect-sites-from-dos-attacks/)

[Install fail2ban to protect your site from DOS attacks](https://www.garron.me/en/go2linux/fail2ban-protect-web-server-http-dos-attack.html)

[How to unban an IP in fail2ban](https://linuxhint.com/unban-ip-fail2ban/)

---

1. You have to set a protection against scans on your VM’s open ports

`sudo apt install postfix && apt install psad`

`sudo lsof -i -P -n | grep LISTEN` - see all open ports

## PSAD

```c
// Status
sudo psad -S

// Unban everyone
sudo psad -F

//Allow particular addresses
sudo psad --fw-rm-block-ip <IP-Address>
```

[](https://www.unixmen.com/how-to-block-port-scan-attacks-with-psad-on-ubuntu-debian/)

---

1. **Stop the services you don’t need for this project.**

```c
// Location of different services
/etc/init.d

// Disable certain service
sudo systemctl disable SERVICE_NAME
```

![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%202.png)

`sudo systemctl list-unit-files --type service | grep enabled`

`sudo service--status-all`

![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%203.png)

---

1. Create a script that updates all the sources of package, then your packages and which logs the whole in a file named /var/log/update_script.log. Create a scheduled
   task for this script once a week at 4AM and every time the machine reboots.

**`sudo crontab -e` -** for setting systemwide cron tasks

```c
#!bin/bash
#
# Updates all source packages. Log saved to /var/log/update_script.log

sudo echo "--------------------------" >> /var/log/update_script.log
sudo echo "Date $(date)" >> /var/log/update_script.log
sudo apt-get update -y >> /var/log/update_script.log
sudo apt-get upgrade -y >> /var/log/update_script.log
echo "--------------------------" >> /var/log/update_script.log
```

[](https://linuxize.com/post/scheduling-cron-jobs-with-crontab/)

### **Cron job syntax**

Crontabs use the following flags for adding and listing cron jobs.

-   **`crontab -e`**: edits crontab entries to add, delete, or edit cron jobs.
-   **`crontab -l`**: list all the cron jobs for the current user.
-   **`crontab -u username -l`**: \*\*\*\*list another user's crons.
-   **`crontab -u username -e`**: \*\*\*\*edit another user's crons.

## Crontab for sudo tasks

[How to run a cron job using the sudo command](https://askubuntu.com/questions/173924/how-to-run-a-cron-job-using-the-sudo-command)

[How to Automate Tasks with cron Jobs in Linux](https://www.freecodecamp.org/news/cron-jobs-in-linux/)

---

1. Make a script to monitor changes of the /etc/crontab file and sends an email to root if it has been modified. Create a scheduled script task every day at midnight.

![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%204.png)

1. `sudo apt install mailutils`
2. `mail` - Will show all messages.

```c
// Test mailservice
echo "Message_text" | mail -s "Subject" root
```

## Crontab tool

[Crontab.guru - The cron schedule expression editor](https://crontab.guru/every-day-at-1am)

### Most simple mail service

[](https://www.cyberciti.biz/faq/delete-all-root-email-mailbox/)

## Local mail service

[Setting Up Local Mail Delivery on Ubuntu with Postfix and Mutt](https://www.cmsimike.com/blog/2011/10/30/setting-up-local-mail-delivery-on-ubuntu-with-postfix-and-mutt/)

---

# Web part

1.  You have to set a web server who should BE available on the VM’s IP or an host
    ([init.login.com](http://init.login.com/) for exemple).
    `sudo ufw app list`

        Apache FULL or WWW full

2.  About the packages of your web server, you can choose
    between ~~Nginx~~ and Apache. You have to set a self-signed SSL on all of your services.
3.  You have to set a web "application" from those choices:
    • A login page.
    • A display site.
    • A wonderful website that blow our minds.
    The web-app COULD be written with any language you want.

# Deployment Part

1. Propose a functional solution for deployment automation.

[How To Create a Self-Signed SSL Certificate for Apache in Debian 10 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-debian-10)

## Apache

-   Verify apache installation
    -   Just write your servers IP to the browser

## SSL setup

## 1. Create SSL certificate

```c
// Creating SSL certificate and the key
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt

// SSL folder
/etc/ssl
```

---

## 2. Apache configs for SSL

> To set up Apache SSL securely, we will be using the recommendations by Remy van Elst on the [Cipherli.st](https://cipherli.st/) site. This site is designed to provide easy-to-consume encryption settings for popular software.

```c
// HSTS disabled for now

/etc/apache2/conf-available/ssl-params.conf

SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLHonorCipherOrder On
# Disable preloading HSTS for now.  You can use the commented out header line that includes
# the "preload" directive if you understand the implications.
# Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
# Requires Apache >= 2.4
SSLCompression off
SSLUseStapling on
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
# Requires Apache >= 2.4.11
SSLSessionTickets O
```

## /etc/apache2/sites-available/default-ssl.conf

-   Backupping orifinal file

    -   `sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/default-ssl.conf.bak`

    1. Editing original file

        1. /etc/ssl/certs/`apache-selfsigned.crt`
        2. /etc/ssl/private/`apache-selfsigned.key`
        3. `ServerName SERVER_IP`

        ![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%205.png)

1. `etc/apache2/sites-available/000-default.conf`

    1. `Redirect "/" "https://your_domain_or_IP/"`

    ## 3. Firewall configs

    1. See ufw app list
        1. `sudo ufw app list`
    2. Check ufw status

        1. `sudo ufw status`
        2. `sudo ufw allow 'WWW Full'`
        3. `sudo ufw delete allow 'WWW'`

        ![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%206.png)

### 3. Enable Apache settings

1. `sudo a2enmod ssl`
2. `sudo a2enmod headers`
3. `sudo a2ensite default-ssl`
4. `sudo a2enconf ssl-params`
5. Test Syntax
    1. `sudo apache2ctl configtest`
6. `sudo systemctl restart apache2`

**ERROR:**

AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message

Fix → `sudo vim /etc/apache2/apache2.conf`

![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%207.png)

1. Connecting to website again

    1. https://SERVER_IP
    2. http://SERVER_IP

    > You should be taken to your site. If you look in the browser address bar, you will see a lock with an “x” over it or another similar “not secure” notice. In this case, this just means that the certificate cannot be validated. It is still encrypting your connection.

    ![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%208.png)

## 6. Permanent redirect

> If your redirect worked correctly and you are sure you want to allow only encrypted traffic, you should modify the unencrypted Apache Virtual Host again to make the redirect permanent.

1.  `sudo vim /etc/apache2/sites-available/000-default.conf`
2.  `Redirect permanent "/" "https://your_domain_or_IP/"`
3.  `sudo apache2ctl configtest`
    1. syntax test
4.  `sudo systemctl restart apache2`

> This will make the redirect permanent, and your site will only serve traffic over HTTPS.

> You have configured your Apache server to use strong encryption for client connections. This will allow you serve requests securely, and will prevent outside parties from reading your traffic.

## Deployment

**File to change website look**

`/var/www/html/index.html`

[How to Create a Simple Login Page Using HTML and CSS](https://www.c-sharpcorner.com/article/creating-a-simple-login-page-using-html-and-css/)

# Notes

## Subnet calculator

[IP Subnet Calculator](https://www.calculator.net/ip-subnet-calculator.html?cclass=any&csubnet=20&cip=10.11.254.254&ctype=ipv4&printit=0&x=0&y=0)

## NAT

-   Feature of the router.
-   Translates private IP addresses to public IP addresses - and vice versa.
-   If every devices of the world would have public IP address, they would
-   In the future not needed, because IPv6 has so many IP addresses

![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%209.png)

## Subnet mask

-   Separates Network portion from the Device portion

![Untitled](roger-skyline-1%20-%20How%20to%20properly%20drink%20shots%20f88bd97217274b7996112dbdb0d6d132/Untitled%2010.png)

## Security

-   Allowing access to root trough internet is NOT SAFE
-   More preferable way is make new user and give that user root privileges

[5 Steps to Secure Linux (protect from hackers)](https://youtu.be/ZhMw53Ud2tY)

[How To Protect Your Linux Server From Hackers!](https://www.youtube.com/watch?v=fKuqYQdqRIs&ab_channel=LiveOverflow)

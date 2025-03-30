<p align="center">
  <img src="https://github.com/maaloum-yassine/42/blob/main/logo_project42/born2beroote.png" alt="Cub3D 42 project badge"/>
</p>

## 🎥 Score 🥇✅
<p align="center">
  <img src="https://github.com/maaloum-yassine/42/blob/main/score/Born2beroot_.png" alt="Score 42 project 125"/>
</p>

---

## 📑 Overview

This project contains step-by-step instructions for configuring a secure Debian virtual machine with:
- Strong password policies
- SSH configuration
- Firewall rules
- User and group management
- Periodic monitoring with cron
- Optional bonus implementations (WordPress with LLMP stack and FTP server)

## 📋 Table of Contents

- [Installation](#-installation)
- [Sudo Configuration](#-sudo-configuration)
- [SSH Setup](#-ssh-setup)
- [User Management](#-user-management)
- [Monitoring Script](#-monitoring-script)
- [Cron Jobs](#-cron-jobs)
- [Bonus Features](#-bonus-features)
  - [LLMP Stack](#llmp-stack)
  - [FTP Server](#ftp-server)
- [Evaluation Checklist](#-evaluation-checklist)
- [Common Issues](#-common-issues)
- [Resources](#-resources)

## 💿 Installation

This guide uses Debian 10 Buster for compatibility with the 42 curriculum requirements.

- [Debian 10 ISO download](https://www.debian.org/download)
- [Installation walkthrough video](https://youtu.be/2w-2MX5QrQw) (no audio)

## 🔐 Sudo Configuration

### Installing sudo
```bash
$ su -
Password: [your root password]
# apt install sudo
# dpkg -l | grep sudo    # Verify installation
```

### Adding User to sudo Group
```bash
# adduser <username> sudo
$ getent group sudo      # Verify user was added
# reboot                 # Apply changes
$ sudo -v                # Verify sudo access
```

### Setting Up sudo Configuration
```bash
$ sudo vi /etc/sudoers.d/sudoconfig
```

Add the required security settings:
```
Defaults        passwd_tries=3
Defaults        badpass_message="Incorrect password! Please try again."
Defaults        logfile="/var/log/sudo/sudo.log"
Defaults        log_input,log_output
Defaults        iolog_dir="/var/log/sudo"
Defaults        requiretty
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

Create the log directory:
```bash
$ sudo mkdir /var/log/sudo
```

## 🔄 SSH Setup

### Installing & Configuring SSH
```bash
$ sudo apt install openssh-server
$ dpkg -l | grep ssh    # Verify installation
$ sudo vi /etc/ssh/sshd_config
```

Update these key settings:
```
Port 4242              # Change from default port 22
PermitRootLogin no     # Disable root login via SSH
```

### Setting Up Firewall (UFW)
```bash
$ sudo apt install ufw
$ sudo ufw enable
$ sudo ufw allow 4242
$ sudo ufw status      # Verify configuration
```

### Connecting via SSH
From your local machine:
```bash
$ ssh <username>@<ip-address> -p 4242
```

## 👤 User Management

### Password Policy Configuration

#### Set Password Age Rules
```bash
$ sudo vi /etc/login.defs
```

Update these lines:
```
PASS_MAX_DAYS   30    # Password expires after 30 days
PASS_MIN_DAYS   2     # Minimum 2 days between password changes
PASS_WARN_AGE   7     # Warning 7 days before expiration
```

#### Configure Password Strength Requirements
```bash
$ sudo apt install libpam-pwquality
$ sudo vi /etc/pam.d/common-password
```

Add these requirements to the pam_pwquality.so line:
```
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

### Creating Groups and Users
```bash
$ sudo addgroup user42
$ sudo adduser <new_username>
$ sudo adduser <new_username> user42
$ getent group user42    # Verify user was added
```

## 📊 Monitoring Script

The monitoring script broadcasts system information to all terminal users every 10 minutes via the `wall` command. It provides essential system metrics and usage statistics.

### Script Content (monitoring.sh)
```bash
#!/bin/bash

# System architecture and kernel version
arc=$(uname -a)

# Number of physical processors
pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l)

# Number of virtual processors
vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)

# RAM usage statistics
fram=$(free -m | awk '$1 == "Mem:" {print $2}')
uram=$(free -m | awk '$1 == "Mem:" {print $3}')
pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')

# Disk usage statistics
fdisk=$(df -Bg | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
udisk=$(df -Bm | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
pdisk=$(df -Bm | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')

# CPU load percentage
cpul=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')

# Last system reboot
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')

# Check if LVM is active
lvmt=$(lsblk | grep "lvm" | wc -l)
lvmu=$(if [ $lvmt -eq 0 ]; then echo no; else echo yes; fi)

# Active TCP connections
# Note: Requires net-tools package (sudo apt install net-tools)
ctcp=$(cat /proc/net/sockstat{,6} | awk '$1 == "TCP:" {print $3}')

# Number of users logged in
ulog=$(users | wc -w)

# Network information (IP and MAC addresses)
ip=$(hostname -I)
mac=$(ip link show | awk '$1 == "link/ether" {print $2}')

# Number of sudo commands executed
cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

# Display all information using wall command
wall "	#Architecture: $arc
	#CPU physical: $pcpu
	#vCPU: $vcpu
	#Memory Usage: $uram/${fram}MB ($pram%)
	#Disk Usage: $udisk/${fdisk}Gb ($pdisk%)
	#CPU load: $cpul
	#Last boot: $lb
	#LVM use: $lvmu
	#Connexions TCP: $ctcp ESTABLISHED
	#User log: $ulog
	#Network: IP $ip ($mac)
	#Sudo: $cmds cmd"
```

### Script Functionality

The monitoring script collects and displays the following information:

1. **Architecture and Kernel Version**: Shows the system's architecture and kernel information
2. **CPU Information**: Displays both physical and virtual CPU counts
3. **Memory Usage**: Shows current RAM usage in MB and percentage
4. **Disk Usage**: Displays storage usage in GB and percentage
5. **Current CPU Load**: Shows the current processor load percentage
6. **Last Reboot**: Indicates when the system was last restarted
7. **LVM Status**: Shows whether Logical Volume Manager is in use
8. **Active TCP Connections**: Displays the count of established TCP connections
9. **User Sessions**: Shows how many users are currently logged in
10. **Network Information**: Displays the server's IP and MAC addresses
11. **Sudo Command Count**: Shows how many sudo commands have been executed

### Setting Up the Script

1. Create the script file:
```bash
$ sudo vi /path/to/monitoring.sh
```

2. Paste the script content

3. Make the script executable:
```bash
$ sudo chmod +x /path/to/monitoring.sh
```

4. Install any required dependencies:
```bash
$ sudo apt install net-tools
```

## ⏱️ Cron Jobs

### Setting Up the Monitoring Script to Run Periodically
```bash
$ sudo crontab -u root -e
```

Add this line to run every 10 minutes:
```
*/10 * * * * sh /path/to/monitoring.sh
```

Check the scheduled jobs:
```bash
$ sudo crontab -u root -l
```

## ✨ Bonus Features

### LLMP Stack

#### Installing Lighttpd
```bash
$ sudo apt install lighttpd
$ sudo ufw allow 80
```

#### Installing MariaDB
```bash
$ sudo apt install mariadb-server
$ sudo mysql_secure_installation
```

#### Installing PHP
```bash
$ sudo apt install php-cgi php-mysql
```

#### Setting Up WordPress
```bash
$ sudo apt install wget
$ sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
$ sudo tar -xzvf /var/www/html/latest.tar.gz
$ sudo cp -r /var/www/html/wordpress/* /var/www/html
$ sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```

Edit the WordPress configuration:
```bash
$ sudo vi /var/www/html/wp-config.php
```

Enable required Lighttpd modules:
```bash
$ sudo lighty-enable-mod fastcgi
$ sudo lighty-enable-mod fastcgi-php
$ sudo service lighttpd force-reload
```

### FTP Server

#### Installing & Configuring FTP
```bash
$ sudo apt install vsftpd
$ sudo ufw allow 21
$ sudo vi /etc/vsftpd.conf
```

Important configuration options:
```
write_enable=YES
chroot_local_user=YES
user_sub_token=$USER
local_root=/home/$USER/ftp
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
```

## ✅ Evaluation Checklist

- [ ] Partition setup with LVM
- [ ] SSH running on port 4242 only
- [ ] UFW configured with port 4242 open
- [ ] Strong password policy implemented
- [ ] User with sudo permissions created
- [ ] sudo configuration as required
- [ ] hostname set correctly
- [ ] user42 group created with appropriate users
- [ ] Monitoring script displaying all required information
- [ ] Script monitoring cron job running every 10 minutes
- [ ] Bonus: Web server with functional WordPress
- [ ] Bonus: FTP service properly configured

## ❓ Common Issues

- **SSH connection refused**: Check if SSH is running with `systemctl status ssh` and verify firewall settings
- **sudo permissions not working**: Verify user is in sudo group with `getent group sudo`
- **Password policy not applying**: Make sure PAM modules are properly configured
- **Web server not accessible**: Check if port 80 is allowed in UFW and Lighttpd is running
- **Monitoring script not displaying**: Ensure the script has execute permissions and all dependencies are installed

## 📚 Resources

- [Debian Documentation](https://www.debian.org/doc/)
- [UFW Manual](https://manpages.debian.org/buster/ufw/ufw.8.en.html)
- [WordPress Installation Guide](https://wordpress.org/support/article/how-to-install-wordpress/)
- [MariaDB Documentation](https://mariadb.com/kb/en/documentation/)
- [Bash Scripting Guide](https://tldp.org/LDP/abs/html/)

---

⭐ Star this repository if you found it helpful!

Created with ❤️ by [Your Name]
---

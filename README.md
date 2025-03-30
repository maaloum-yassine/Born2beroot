# 🚀 Born2beroot



## 📑 Overview

This projet contains step-by-step instructions for configuring a secure Debian virtual machine with:
Strong password policies
SSH configuration
Firewall rules
User and group management
Periodic monitoring with cron
Optional bonus implementations (WordPress with LLMP  stack ( LLMP = Linux + Lighttpd + MySQL/MariaDB + PHP) and FTP server)


## 📋 Table of Contents

- [Installation](#-installation)
- [Sudo Configuration](#-sudo-configuration)
- [SSH Setup](#-ssh-setup)
- [User Management](#-user-management)
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

## ⏱️ Cron Jobs

### Setting Up the Monitoring Script
```bash
$ sudo crontab -u root -e
```

Add this line to run every 10 minutes:
```
*/10 * * * * sh /path/to/monitoring.sh
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
- [ ] Script monitoring cron job running
- [ ] Bonus: Web server with functional WordPress
- [ ] Bonus: FTP service properly configured

## ❓ Common Issues

- **SSH connection refused**: Check if SSH is running with `systemctl status ssh` and verify firewall settings
- **sudo permissions not working**: Verify user is in sudo group with `getent group sudo`
- **Password policy not applying**: Make sure PAM modules are properly configured
- **Web server not accessible**: Check if port 80 is allowed in UFW and Lighttpd is running

## 📚 Resources

- [Debian Documentation](https://www.debian.org/doc/)
- [UFW Manual](https://manpages.debian.org/buster/ufw/ufw.8.en.html)
- [WordPress Installation Guide](https://wordpress.org/support/article/how-to-install-wordpress/)
- [MariaDB Documentation](https://mariadb.com/kb/en/documentation/)

---

⭐ Star this repository if you found it helpful!

Created with ❤️ by [Your Name]

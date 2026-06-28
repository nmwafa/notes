# Linux SysAdmin Cheatsheet

Essential Commands, Scripts & Troubleshooting Guide

> Credit: Zubair H

## Table of Contents

- [Quick Start Guide](#quick-start-guide)
- [File System Management](#file-system-management)
  - [Basic File Operations](#basic-file-operations)
  - [Disk & Storage Management](#disk--storage-management)
  - [Permissions & Ownership](#permissions--ownership)
- [User & Group Administration](#user--group-administration)
  - [User Management](#user-management)
  - [Group Management](#group-management)
  - [Sudo Security](#sudo-security)
- [Process Management](#process-management)
  - [Process Monitoring](#process-monitoring)
  - [System Monitoring](#system-monitoring)
  - [Systemd Service Management](#systemd-service-management)
- [Network Configuration](#network-configuration)
- [SSH & Remote Access](#ssh--remote-access)
- [Firewall Management](#firewall-management)
  - [UFW (Ubuntu)](#ufw-ubuntu)
  - [Firewalld (RHEL/CentOS)](#firewalld-rhelcentos)
- [Security & Monitoring](#security--monitoring)
  - [System Monitoring (Security)](#system-monitoring-security)
  - [Log Management](#log-management)
- [Backup & Recovery](#backup--recovery)
  - [File Backups](#file-backups)
  - [Database Backups](#database-backups)
  - [Critical Security Practices](#critical-security-practices)
- [Shell Scripting Essentials](#shell-scripting-essentials)
- [Cron Jobs](#cron-jobs)
- [Useful One‑Liners](#useful-oneliners)
- [Troubleshooting Guide](#troubleshooting-guide)
  - [SSH Connectivity Issues](#ssh-connectivity-issues)
  - [High CPU Usage](#high-cpu-usage)
  - [Disk Full Issues](#disk-full-issues)
  - [Emergency Recovery](#emergency-recovery)
- [Most Used Commands](#most-used-commands)

---

## Quick Start Guide

This cheatsheet is organized by topic with practical examples. Use the table of contents to quickly navigate to the sections most relevant to your current task.

---

## File System Management

### Basic File Operations

| Command | Description | Example |
|---------|-------------|---------|
| `ls` | List directory contents | `ls -la /home` |
| `cp` | Copy files and directories | `cp -r source_dir/ dest_dir/` |
| `mv` | Move/rename files | `mv oldname newname` |
| `rm` | Remove files/directories | `rm -rf directory/` |
| `find` | Search for files | `find / -name "*.log"` |

### Disk & Storage Management

```bash
# Show disk space usage
df -h

# Show directory sizes
du -sh /var/*

# Find large files (>100MB)
find / -type f -size +100M -exec ls -lh {} \;

# Monitor disk I/O
iostat -x 1
```

### Permissions & Ownership

```bash
# Numeric method
chmod 755 script.sh

# Symbolic method
chmod u+x,g-w,o-r file

# Recursive
chmod -R 644 /path/

# Change owner
chown user file

# Change group
chgrp group file

# Change both
chown user:group file

# Recursive ownership
chown -R user:group /path/
```

---

## User & Group Administration

### User Management

| Command | Description | Example |
|---------|-------------|---------|
| `useradd` | Create new user | `useradd -m -s /bin/bash john` |
| `usermod` | Modify user account | `usermod -aG sudo john` |
| `userdel` | Delete user | `userdel -r john` |
| `passwd` | Change password | `passwd john` |

### Group Management

```bash
# Create new group
groupadd developers

# Add user to group
usermod -aG developers john

# List user groups
groups john

# Remove user from group
gpasswd -d john developers

# View group members
getent group developers
```

### Sudo Security

> **Be cautious when granting sudo privileges.** Use specific commands rather than `ALL` when possible.

```bash
# Edit sudoers file safely
visudo

# Example sudoers entry
john ALL=(ALL) /usr/bin/systemctl, /usr/bin/apt

# Run command as another user
sudo -u john whoami
```

---

## Process Management

### Process Monitoring

| Command | Description | Usage |
|---------|-------------|-------|
| `ps` | Process status | `ps aux | grep nginx` |
| `top` | Interactive process viewer | `top -u www-data` |
| `htop` | Enhanced top | `htop` |
| `kill` | Terminate process | `kill -9 1234` |
| `pkill` | Kill by name | `pkill -f nginx` |

### System Monitoring

```bash
# CPU usage
mpstat 1

# Memory usage
free -h

# I/O statistics
iostat -x 1

# Network traffic
iftop

# Real-time monitoring
watch -n 1 'df -h; echo; free -h'

# Find process using port
lsof -i :80

# Show process tree
pstree -p

# Nice value (priority)
nice -n 10 command
renice 15 -p 1234

# Background jobs
jobs
fg %1
bg %1
```

### Systemd Service Management

```bash
# Start/stop service
sudo systemctl start nginx
sudo systemctl stop nginx

# Enable/disable service
sudo systemctl enable nginx
sudo systemctl disable nginx

# Check service status
sudo systemctl status nginx

# Reload service config
sudo systemctl reload nginx

# View service logs
sudo journalctl -u nginx -f
```

---

## Network Configuration

| Command | Description | Example |
|---------|-------------|---------|
| `ip` | Modern network config | `ip addr show` |
| `ifconfig` | Legacy network config | `ifconfig eth0` |
| `netstat` | Network statistics | `netstat -tulpn` |
| `ss` | Socket statistics | `ss -tulpn` |
| `ping` | Network connectivity | `ping google.com` |

---

## SSH & Remote Access

```bash
# Basic SSH connection
ssh user@hostname

# SSH with specific port
ssh -p 2222 user@hostname

# SSH with key authentication
ssh -i ~/.ssh/key.pem user@hostname

# SSH tunnel
ssh -L 8080:localhost:80 user@hostname

# SCP file transfer
scp file.txt user@hostname:/path/
scp user@hostname:/path/file.txt .
```

---

## Firewall Management

### UFW (Ubuntu)

```bash
# Enable firewall
sudo ufw enable

# Allow SSH
sudo ufw allow ssh

# Allow specific port
sudo ufw allow 8080/tcp

# Deny port
sudo ufw deny 25/tcp

# Status
sudo ufw status verbose
```

### Firewalld (RHEL/CentOS)

```bash
# Start service
sudo systemctl start firewalld

# Allow service
sudo firewall-cmd --add-service=http

# Allow port
sudo firewall-cmd --add-port=8080/tcp

# Make permanent
sudo firewall-cmd --runtime-to-permanent

# List rules
sudo firewall-cmd --list-all
```

---

## Security & Monitoring

### System Monitoring (Security)

```bash
# Monitor system resources
top
htop

# Disk I/O monitoring
iotop

# Network monitoring
nethogs

# Continuous log monitoring
tail -f /var/log/syslog

# Monitor failed login attempts
sudo lastb

# Check for rootkits
sudo rkhunter --check
```

### Log Management

| Command | Description | Usage |
|---------|-------------|-------|
| `journalctl` | Systemd journal | `journalctl -f` |
| `tail` | Follow log files | `tail -f /var/log/nginx/access.log` |
| `grep` | Search logs | `grep "ERROR" /var/log/syslog` |
| `logrotate` | Manage log files | `logrotate -f /etc/logrotate.conf` |

---

## Backup & Recovery

### File Backups

```bash
# Simple tar backup
tar -czf backup-$(date +%Y%m%d).tar.gz /important/data

# Rsync backup
rsync -av /source/ /backup/destination/

# Incremental backup with rsync
rsync -av --link-dest=/previous/backup /source/ /new/backup/
```

### Database Backups

```bash
# MySQL backup
mysqldump -u user -p database > backup.sql

# PostgreSQL backup
pg_dump database > backup.sql

# MongoDB backup
mongodump --host localhost --db mydb --out /backup/
```

### Critical Security Practices

> Always test your backups regularly. Store backups in multiple locations including off-site. Encrypt sensitive backup data.

---

## Shell Scripting Essentials

### Basic Script Template

```bash
#!/bin/bash
# Script: system_info.sh
# Description: Basic system information script

set -e  # Exit on error

# Variables
HOSTNAME=$(hostname)
DATE=$(date +%Y-%m-%d)
LOG_FILE="/var/log/system_info.log"

# Functions
check_disk_usage() {
    echo "== Disk Usage =="
    df -h | grep -v tmpfs
}

check_memory() {
    echo "== Memory Usage =="
    free -h
}

# Main script
main() {
    echo "System Report for $HOSTNAME - $DATE"
    check_disk_usage
    check_memory
}

# Execute main function
main "$@"
```

---

## Cron Jobs

### Cron Syntax

```
# Minute Hour Day Month DayOfWeek Command
# * * * * * command

# Examples:
# Every minute
* * * * * /path/command

# Every day at 2:30 AM
30 2 * * * /path/backup.sh

# Every Monday at 6 PM
0 18 * * 1 /path/script.sh

# Every 10 minutes
*/10 * * * * /path/monitor.sh
```

### Cron Management

```bash
# Edit user crontab
crontab -e

# List current crontab
crontab -l

# Remove all cron jobs
crontab -r

# System cron (root)
sudo crontab -e

# Cron directories
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

---

## Useful One‑Liners

```bash
# Find and delete old log files
find /var/log -name "*.log" -mtime +30 -delete

# Count lines of code in project
find . -name "*.py" -exec wc -l {} + | tail -1

# Monitor multiple log files simultaneously
tail -f /var/log/nginx/*.log /var/log/mysql/*.log

# Create backup with timestamp
tar -czf "backup-$(date +%Y%m%d-%H%M%S).tar.gz" /important/data

# Kill processes matching pattern
pkill -f "python3 app.py"

# SSH tunnel for MySQL access
ssh -L 3306:localhost:3306 user@dbserver
```

---

## Troubleshooting Guide

### SSH Connectivity Issues

```bash
# Check if SSH service is running
sudo systemctl status ssh

# Verify firewall rules
sudo ufw status

# Check SSH port
sudo netstat -tulpn | grep :22

# Examine SSH logs
sudo journalctl -u ssh -f

# Test from different network
```

### High CPU Usage

```bash
# Identify top processes
top
htop

# Check for zombie processes
ps aux | grep defunct

# Monitor system load
uptime

# Check for runaway scripts
ps aux --sort=-%cpu | head

# Examine application logs
```

### Disk Full Issues

```bash
# Check disk usage
df -h

# Find large directories
du -sh /* | sort -hr

# Check for large log files
find /var/log -type f -size +100M

# Clear package cache
sudo apt clean   # or sudo yum clean all

# Check for core dumps
find / -name "core" -size +10M
```

### Emergency Recovery

> **These commands should only be used when the system is unresponsive or in critical condition.**

```bash
# Force filesystem check on reboot
sudo touch /forcefsck

# Enter single-user mode (emergency)
sudo systemctl rescue

# Kill process by name (forceful)
pkill -9 process_name

# Remount filesystem as read-write
mount -o remount,rw /

# Emergency memory cleanup
echo 3 > /proc/sys/vm/drop_caches

# Force service restart
sudo systemctl reset-failed service_name
```

---

## Most Used Commands

### File Operations

| Command | Description |
|---------|-------------|
| `ls -la` | Detailed listing |
| `cp -r` | Recursive copy |
| `rm -rf` | Force remove |
| `find / -name` | Search files |
| `grep -r "text"` | Recursive search |

### Networking

| Command | Description |
|---------|-------------|
| `ip addr` | Network interfaces |
| `netstat -tulpn` | Open ports |
| `ss -tulpn` | Modern netstat |
| `ping` | Network test |
| `traceroute` | Route tracing |

### System Info

| Command | Description |
|---------|-------------|
| `uname -a` | System info |
| `df -h` | Disk space |
| `free -h` | Memory usage |
| `uptime` | System uptime |
| `lscpu` | CPU information |

### Process Management

| Command | Description |
|---------|-------------|
| `ps aux` | All processes |
| `top` | Process monitor |
| `kill -9` | Force kill |
| `pkill` | Kill by name |
| `nice` | Process priority |

### Keyboard Shortcuts

| Shortcut | Description | Context |
|----------|-------------|---------|
| `Ctrl + C` | Interrupt/Kill process | Terminal |
| `Ctrl + Z` | Suspend process | Terminal |
| `Ctrl + D` | Exit shell | Terminal |
| `Ctrl + R` | Search command history | Bash |
| `Ctrl + A` | Move to line start | Terminal |
| `Ctrl + E` | Move to line end | Terminal |

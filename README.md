# Server Security Implementation Guide

This guide outlines the steps to secure your server by monitoring login activity, implementing IP blacklisting, geofencing, account lockout mechanisms, and deploying an Intrusion Detection and Prevention System (IDPS).

## 1. Monitor Login Activity and Logs

### Check and Configure Logs
1. On **Linux**:
   - Debian-based: Check `/var/log/auth.log`
   - RHEL/CentOS-based: Check `/var/log/secure`
2. Ensure **SSH** service logs all activity:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   Confirm the following:
   ```bash
   LogLevel VERBOSE
   ```

### Install Log Monitoring Tool
1. Install **fail2ban**:
   ```bash
   sudo apt-get install fail2ban    # Debian/Ubuntu
   sudo yum install fail2ban        # CentOS/RHEL
   ```

### Enable Login Alerts (Optional)
You can receive email alerts for logins:
1. Add this to `/etc/profile`:
   ```bash
   echo 'ALERT - `hostname` - Login detected' | mail -s "Login Alert" youremail@domain.com
   ```

## 2. IP Blacklisting and Geofencing

### IP Blacklisting using Fail2ban
1. Edit the `jail.local` file:
   ```bash
   sudo nano /etc/fail2ban/jail.local
   ```
2. Add the following under `[sshd]`:
   ```ini
   [sshd]
   enabled = true
   filter = sshd
   action = iptables-multiport[name=SSH, port="ssh", protocol=tcp]
   logpath = /var/log/auth.log
   maxretry = 5
   bantime = 86400  # 1 day
   ```

### Geofencing using GeoIP and Iptables
1. Install **xtables-addons** and GeoIP:
   ```bash
   sudo apt-get install xtables-addons-common libtext-csv-xs-perl libgeoip-dev
   sudo geoipupdate
   ```

2. Block specific countries by editing iptables rules:
   ```bash
   sudo iptables -A INPUT -m geoip --src-cc CN,RU -j DROP  # Blocks China and Russia
   ```

## 3. Account Lockout Mechanism

### Fail2ban Lockout Configuration
1. Configure **fail2ban** in `/etc/fail2ban/jail.local`:
   ```ini
   maxretry = 5
   bantime = 3600  # Lockout for 1 hour after 5 failed attempts
   ```

### Linux PAM Account Lockout (Optional)
1. Edit `/etc/pam.d/sshd`:
   ```bash
   sudo nano /etc/pam.d/sshd
   ```
2. Add the following line:
   ```bash
   auth required pam_tally2.so deny=5 onerr=fail unlock_time=600
   ```

### Test Lockout
1. Test by attempting wrong logins to ensure the account locks after the set number of failures.

## 4. Deploy Intrusion Detection and Prevention System (IDPS)

### Install and Configure Snort (IDS)
1. Install **Snort**:
   ```bash
   sudo apt-get install snort
   ```
2. Configure Snort:
   ```bash
   sudo nano /etc/snort/snort.conf
   ```
   Set your network:
   ```bash
   var HOME_NET 192.168.1.0/24
   ```

### Install and Configure Suricata (IPS)
1. Install **Suricata**:
   ```bash
   sudo apt-get install suricata
   ```
2. Run Suricata in IPS mode:
   ```bash
   sudo suricata -c /etc/suricata/suricata.yaml -i eth0
   ```

### Review Logs
1. Monitor logs for activity in `/var/log/snort` (for Snort) and `/var/log/suricata` (for Suricata).

## Summary
- **Monitor login activity** using system logs and Fail2ban.
- **Implement IP blacklisting** and **geofencing** using Fail2ban and iptables with GeoIP.
- **Configure account lockout** with Fail2ban or PAM.
- **Deploy IDPS** using Snort for detection and Suricata for prevention.

This guide will help secure your server from brute force attacks, unauthorized logins, and other security threats.
```

### Review Notes:
- **Fail2ban** is used for both login monitoring and enforcing lockouts.
- **IP blacklisting** and **geofencing** are managed with **iptables** and **GeoIP**.
- **Snort** and **Suricata** handle intrusion detection and prevention.
- This guide ensures proper configuration to prevent future attacks.

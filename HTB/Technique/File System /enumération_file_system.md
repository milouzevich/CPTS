**Fichiers clés par type de serveur**

---

## Web - Apache / Nginx

```
/etc/apache2/apache2.conf
/etc/apache2/sites-enabled/000-default.conf
/etc/apache2/sites-available/*.conf
/etc/nginx/nginx.conf
/etc/nginx/sites-enabled/default
/etc/nginx/conf.d/*.conf

/var/www/html/config.php
/var/www/html/.env
/var/www/html/wp-config.php        # WordPress
/var/www/html/configuration.php    # Joomla
/var/www/html/config/database.php  # Laravel

/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/nginx/access.log
/var/log/nginx/error.log

/etc/hosts
/etc/hostname
```
```
# Emplacements courants de httpd.conf

/etc/httpd/conf/httpd.conf          # CentOS / RHEL / Fedora
/etc/apache2/apache2.conf           # Debian / Ubuntu (équivalent)
/usr/local/apache2/conf/httpd.conf  # Compilation depuis les sources
/usr/local/etc/apache24/httpd.conf  # FreeBSD
/opt/lampp/etc/httpd.conf           # XAMPP
C:\xampp\apache\conf\httpd.conf     # XAMPP Windows
C:\Apache24\conf\httpd.conf         # Windows natif
```

**Via LFI**
```
php://filter/convert.base64-encode/resource=/etc/httpd/conf/httpd.conf
php://filter/convert.base64-encode/resource=/etc/apache2/apache2.conf
```

**Ce qu'on cherche dedans**
```
DocumentRoot     → racine du site web
ServerName       → vhosts / domaines
CustomLog        → chemin exact des access.log
ErrorLog         → chemin exact des error.log
Include          → autres fichiers de config chargés
```
---

## PHP

```
/etc/php/7.4/apache2/php.ini
/etc/php/8.1/apache2/php.ini
/etc/php/*/cli/php.ini

# Sessions (log poisoning)
/tmp/sess_PHPSESSID
/var/lib/php/sessions/sess_PHPSESSID
```

---

## SSH

```
/etc/ssh/sshd_config
/home/USER/.ssh/id_rsa
/home/USER/.ssh/id_ecdsa
/home/USER/.ssh/authorized_keys
/root/.ssh/id_rsa
/root/.ssh/authorized_keys
/etc/ssh/ssh_host_rsa_key
```

---

## Base de données

```
# MySQL
/etc/mysql/mysql.conf.d/mysqld.cnf
/etc/mysql/my.cnf
/var/lib/mysql/mysql/user.MYD
/root/.mysql_history

# PostgreSQL
/etc/postgresql/*/main/postgresql.conf
/etc/postgresql/*/main/pg_hba.conf

# SQLite
/var/db/*.db
/opt/APP/*.db
/home/USER/*.db
```

---

## Système Linux (toujours)

```
/etc/passwd
/etc/shadow
/etc/group
/etc/sudoers
/etc/crontab
/etc/cron.d/*
/var/spool/cron/crontabs/root

/proc/version
/proc/cmdline
/proc/self/environ
/proc/self/cmdline
/proc/self/maps
/proc/net/tcp
/proc/net/fstat

/home/USER/.bashrc
/home/USER/.bash_history
/root/.bash_history
/root/.bashrc

/var/log/auth.log
/var/log/syslog
/var/log/kern.log
/var/log/mail.log
/var/log/cron.log
```

---

## Applications Java / Spring Boot

```
/opt/APP/application.properties
/opt/APP/application.yml
/opt/APP/config/application.properties
/opt/APP/*.jar   (→ unzip pour lire)
/home/APP/.env

# Actuator endpoints (HTTP)
/actuator/env
/actuator/heapdump
/actuator/mappings
```

---

## Docker / Kubernetes

```
/.dockerenv
/etc/docker/daemon.json
/run/secrets/*
/var/run/docker.sock

# K8s
/run/secrets/kubernetes.io/serviceaccount/token
/run/secrets/kubernetes.io/serviceaccount/ca.crt
/var/run/secrets/kubernetes.io/serviceaccount/namespace
```

---

## FTP / Mail

```
/etc/vsftpd.conf
/etc/proftpd/proftpd.conf
/etc/pure-ftpd/pure-ftpd.conf

/etc/postfix/main.cf
/etc/exim4/exim4.conf.template
/var/mail/USER
/var/spool/mail/USER
```

---

## Credentials & Configs génériques

```
/etc/environment
/.env
/opt/APP/.env
/opt/APP/config.yml
/opt/APP/settings.py    # Django
/opt/APP/settings.json
/opt/APP/config.json
/opt/APP/secrets.yml
/srv/APP/config*
/usr/local/etc/APP/*
```

# **Fichiers clés - Windows**

---

## Système & Credentials

```
C:\Windows\System32\drivers\etc\hosts
C:\Windows\System32\drivers\etc\networks
C:\Windows\win.ini
C:\Windows\system.ini

# SAM & hashes (nécessite SYSTEM)
C:\Windows\System32\config\SAM
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\config\SECURITY
C:\Windows\System32\config\SOFTWARE

# Backup SAM (souvent lisible)
C:\Windows\Repair\SAM
C:\Windows\Repair\SYSTEM
C:\Windows\System32\config\RegBack\SAM
C:\Windows\System32\config\RegBack\SYSTEM
```

---

## Utilisateurs

```
C:\Users\USER\Desktop\*
C:\Users\USER\Documents\*
C:\Users\USER\Downloads\*
C:\Users\USER\.ssh\id_rsa
C:\Users\USER\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
C:\Users\USER\AppData\Local\Microsoft\Windows\History\

# Admin
C:\Users\Administrator\Desktop\*
C:\Users\Administrator\.ssh\id_rsa
```

---

## Logs

```
C:\Windows\System32\winevt\Logs\Security.evtx
C:\Windows\System32\winevt\Logs\System.evtx
C:\Windows\System32\winevt\Logs\Application.evtx
C:\Windows\debug\NetSetup.log
C:\Windows\Panther\unattend.xml        # creds en clair !
C:\Windows\Panther\Unattend\unattend.xml
C:\Windows\System32\Sysprep\unattend.xml
```

---

## IIS (Web Server Windows)

```
C:\inetpub\wwwroot\web.config
C:\inetpub\wwwroot\*.config
C:\Windows\System32\inetsrv\config\applicationHost.config
C:\inetpub\logs\LogFiles\W3SVC1\*

# Apps
C:\inetpub\wwwroot\App_Data\*.mdf
C:\inetpub\wwwroot\connectionStrings.config
```

---

## Active Directory

```
C:\Windows\NTDS\ntds.dit              # DB AD (hashes tous users)
C:\Windows\System32\config\SYSTEM     # clé déchiffrement ntds.dit

# GPO / Scripts
C:\Windows\SYSVOL\sysvol\DOMAIN\Policies\*
C:\Windows\SYSVOL\sysvol\DOMAIN\scripts\*

# Groups.xml (creds GPP - MS14-025)
C:\Windows\SYSVOL\*\Policies\*\MACHINE\Preferences\Groups\Groups.xml
C:\ProgramData\Microsoft\Group Policy\History\*\Groups.xml
```

---

## Applications courantes

```
# WinSCP / PuTTY (creds SSH stockés)
HKCU\Software\SimonTatham\PuTTY\Sessions\*
HKCU\Software\Martin Prikryl\WinSCP 2\Sessions\*

# FileZilla
C:\Users\USER\AppData\Roaming\FileZilla\sitemanager.xml
C:\Users\USER\AppData\Roaming\FileZilla\recentservers.xml

# Navigateurs
C:\Users\USER\AppData\Local\Google\Chrome\User Data\Default\Login Data
C:\Users\USER\AppData\Roaming\Mozilla\Firefox\Profiles\*\logins.json

# VNC
C:\Program Files\RealVNC\vnc4\WinVNC4.exe   (→ reg query pour mot de passe)

# MSSQL
C:\Program Files\Microsoft SQL Server\MSSQL*\MSSQL\DATA\master.mdf
```

---

## PowerShell History (gold mine)

```
C:\Users\USER\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

# Via cmd
type %APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

---

## Registre (creds stockés)

```
# AutoLogon
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

# Creds sauvegardés
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
cmdkey /list

# Services avec creds
reg query "HKLM\SYSTEM\CurrentControlSet\Services" /s | findstr "Password"
```

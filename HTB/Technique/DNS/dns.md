# Enumération 
## Port 53 : 

indentification dans la sortie de nmap
```
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
```

Recherche DNS, pour domaine, sous-domaine ou transfert de zone

recherche du nom de domaine
```bash 
nslookup 10.129.227.211 10.129.227.211
211.227.129.10.in-addr.arpa     name = ns1.cronos.htb.
```
Intégration dans le fichier hosts 
```shell
sudo sh -c  'echo "10.129.227.211     ns1.cronos.htb cronos.htb" >> /etc/hosts'
```

Tentative de transfert de zone
```bash
dig @[serveur_dns] [domaine] -t axfr
dig @ns1.cronos.htb cronos.htb -t axfr
```

```
dig @ns1.cronos.htb cronos.htb -t axfr

; <<>> DiG 9.20.27-2-Debian <<>> @ns1.cronos.htb cronos.htb -t axfr
; (1 server found)
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 84 msec
;; SERVER: 10.129.227.211#53(ns1.cronos.htb) (TCP)
;; WHEN: Thu Oct 01 15:18:35 EDT 2026
;; XFR size: 7 records (messages 1, bytes 203)

```

Ajout des sorties dans le fichier hosts
```bash 
sudo nano /etc/hosts
10.129.227.211     ns1.cronos.htb cronos.htb admin.cronos.htb www.cronos.htb
```


Recherche de sous-domaine
```bash 
gobuster dns --domain cronos.htb -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --no-error
ffuf -u http://10.129.237.241 -H "Host: FUZZ.planning.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -ac
gobuster vhost -u silentium.htb -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt  --append-domain
```

``
```bash 
┌──(kali㉿kali)-[~/…/HTB/LABS/MACHINES/02-MEDIUM]
└─$ gobuster dns --domain cronos.htb -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --no-error          
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     cronos.htb
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
www.cronos.htb ::ffff:10.129.227.211
ns1.cronos.htb ::ffff:10.129.227.211
admin.cronos.htb ::ffff:10.129.227.211

```


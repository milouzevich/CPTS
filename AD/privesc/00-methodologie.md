# Methodologie pour Privesc une machine

### Identifier le user
```
whoami
whoami /groups
whoami /priv
net user <USER>
```
Puis analyse du user dans le chemin relationnel avec le bllodhound

Analyser les fichiers : logs, xmml, .conf 
Pour y trouver des identifiants de connection 

### Winpeas pour analyse automatisée
ref : https://github.com/peass-ng/PEASS-ng




# Tools d'investigation

PowerView.ps1:
-> https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1

Snaffler : 
-> https://github.com/SnaffCon/Snaffler/releases/tag/1.0.244


Inveigh : effectue des attaques par usurpation d'identité et capture des hachages/identifiants via l'analyse de paquets et des écouteurs/sockets spécifiques au protocole.
->  https://github.com/Kevin-Robertson/Inveigh

SharpHound : 
-> https://github.com/SpecterOps/BloodHound-Legacy/tree/master/Collectors

PrintSpoofer :
-> https://github.com/itm4n/PrintSpoofer/releases/tag/v1.0

Lazagne: 
-> https://github.com/alessandroz/lazagne


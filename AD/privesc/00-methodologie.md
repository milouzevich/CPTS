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



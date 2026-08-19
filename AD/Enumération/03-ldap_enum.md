## Ldapsearch
https://notes.benheater.com/books/active-directory/page/ldapsearch
https://devconnected.com/how-to-search-ldap-using-ldapsearch-examples/
https://medium.com/@rajkumarkumawat/%EF%B8%8F-%EF%B8%8F-hacking-lab-tutorial-ldap-enumeration-the-ultimate-guide-for-ethical-hackers-07ede1ecb474


## Sans Credentials
```bash 
# Vérifier si le bind anonyme est autorisé
ldapsearch -x -H ldap://10.129.38.105 -s base namingContexts
ldapsearch -x -H ldap://10.129.38.105 -D "" -w "" -s base namingContexts
```

Si cela fonctionne, tu obtiendras quelque chose comme :

```txt
namingContexts: DC=sequel,DC=htb
```


```bash 
# Lister tous les objets
ldapsearch -x -H ldap://10.129.38.105 -b "DC=sequel,DC=htb"
```

```bash 
#lister les ustilisateurs
ldapsearch -x -H ldap://10.129.38.105 -b "DC=sequel,DC=htb" "(objectClass=user)"
```

```bash 
# lister les groupes
ldapsearch -x -H ldap://10.129.38.105 -b "DC=sequel,DC=htb" "(objectClass=group)"
```

```bash 
# lister les ordinateurs
ldapsearch -x -H ldap://10.129.38.105 -b "DC=sequel,DC=htb" "(objectClass=computer)"
```


Création d'un filtre  
```plain
(&(attribut1=valeur1)(attribut2=valeur2))
```


```bash 
# lister les users avec un filtre sur les sAmAccountName
ldapsearch -x -H ldap://10.129.234.71  -b "DC=baby,DC=vl" "(&(objectCategory=person)(objectClass=user))" sAMAccountName
```

```bash 
# lister les dishintingName
ldapsearch -x -b "dc=baby, dc=vl" "*" -H ldap://BabyDC.baby.vl | grep dn
```
```
netexec ldap BABYDC.baby.vl -u '' -p '' --query "(sAMAccountName=*)" ""
```

```bash
impacket-GetADUsers <DOMAIN_NAME>/ -dc-ip <IP_DC> -debug
```


### Avec des credentials

```bash 
# AVoir toute la liste complète
ldapsearch -D 'Julia.Wong' -w 'Computer1' -H ldap://10.129.44.190 -b "DC=breach,DC=vl" "(objectClass=user)"

# Avoir uniquement les informations essentiels
ldapsearch -D 'Julia.Wong@breach.vl' -w 'Computer1' -H ldap://10.129.44.190 -b "DC=breach,DC=vl" "(&(objectCategory=person)(objectClass=user))" sAMAccountName description memberOf userPrincipalName servicePrincipalName Description
```
```bash 
# trouver liste des users
ldapsearch -x -H 'ldap://10.10.10.100' -D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' -b "dc=active,dc=htb" -s sub "(&(objectCategory=person)(objectClass=user)(! (useraccountcontrol:1.2.840.113556.1.4.803:=2)))" samaccountname | grep sAMAccountName
# trouver liste des users avec impacket
impacket-GetADUsers -all active.htb/svc_tgs -dc-ip 10.10.10.100
#trouver les comptes kerberoastable
ldapsearch -x -H 'ldap://10.10.10.100' -D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' -b "dc=active,dc=htb" -s sub "(&(objectCategory=person)(objectClass=user)(! (useraccountcontrol:1.2.840.113556.1.4.803:=2))(serviceprincipalname=*/*))" serviceprincipalname | grep -B 1 servicePrincipalName
# trouver liste des comptes SPN avec impacket
 impacket-GetUserSPNs active.htb/svc_tgs -dc-ip 10.10.10.100
```



ref : https://github.com/CravateRouge/bloodyAD/wiki/Enumeration
https://www.hackingarticles.in/active-directory-penetration-testing-with-bloodyad/


```bash 
# Enumeration de base LDAP
bloodyad --host 10.129.232.88 -d fluffy.htb -u p.agila -p 'prometheusx-303' get children --otype useronly
bloodyad --host 10.129.232.88 -d fluffy.htb -u p.agila -p 'prometheusx-303' get children --otype computer
bloodyad --host 10.129.41.212 -d administrator.htb -u 'Olivia' -p 'ichliebedich' get children --otype group
bloodyad --host 10.129.232.88 -d fluffy.htb -u p.agila -p 'prometheusx-303' get children --otype container

# Analyse Sépcifique du user
bloodyad --host 10.129.232.88 -d fluffy.htb -u p.agila -p 'prometheusx-303' get object Administrator
### Connaitre les droits du users
bloodyad --host 10.129.228.253 -d sequel.htb -u Ryan.Cooper -p 'NuclearMosquito3' get writable
# Connaitre le group pour user demandé
bloodyad --host 10.129.232.88 -d fluffy.htb -u p.agila -p 'prometheusx-303' get membership p.agila
# Connaitre les membres rattrachés au groupe
bloodyad --host 10.129.232.88 -d fluffy.htb -u p.agila -p 'prometheusx-303' get object "Domain Admins" --attr member
```

Retrouver des comptes SPN
```bash 
# Recherche les comptes avec SPN
netexec ldap DC_IP -u '' -p '' --query "(servicePrincipalName=*)" "sAMAccountName servicePrincipalName"
```


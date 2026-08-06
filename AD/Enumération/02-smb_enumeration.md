```
SMB
 │
 ├── users
 ├── groups
 ├── shares
 ├── sessions
 ├── password policy
 └── RID enumeration
```

```bash 
# test des connexions 
netexec smb 10.10.10.10 -u '' -p ''
netexec ldap 10.10.10.10 -u '' -p ''
netexec winrm 10.10.10.10 -u '' -p ''

# Faire en anonymous et en guest
netexec smb 10.10.10.10 -u '' -p '' --users
netexec smb 10.10.10.10 -u '' -p '' --groups
netexec smb 10.10.10.10 -u '' -p '' --rid-brute
netexec smb 10.10.10.10 -u '' -p '' --rid-brute | grep SidTypeUser | cut -d'\' -f2 | cut -d' ' -f1 | tee users.txt                          
netexec smb 10.10.10.10 -u '' -p '' --shares
netexec smb 10.10.10.10 -u '' -p '' --pass-pol

impacket-lookupsid 'cicada.htb/guest'@cicada.htb -no-pass
impacket-lookupsid 'cicada.htb/guest'@cicada.htb -no-pass | grep 'SidTypeUser' | sed 's/.*\\\(.*\) (SidTypeUser)/\1/' > users.txt

# Consulter les partages 
netexec smb 10.10.10.10 -u '' -p '' --spider HR --regex "."
netexec smb 10.10.10.10 -u '' -p '' --share HR --get-file "Notice from HR.txt" "Notice from HR.txt"


```

# connection sur le service 
```
smbclient -N //<IP>/<RESSOURCE>
smbclient -U <USER> //<IP>/<RESSOURCE>
# commande help, ls, get et put pour la manipulation des fichiers

```

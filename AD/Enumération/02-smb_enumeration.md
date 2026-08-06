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
netexec smb 10.10.10.10 -u '' -p '' --rid-brute | grep SidTypeUser | cut -d'\' -f2 | cut -d' ' -f1 | tee users                            
netexec smb 10.10.10.10 -u '' -p '' --shares
netexec smb 10.10.10.10 -u '' -p '' --pass-pol

# Consulter les partages 
netexec smb 10.10.10.10 -u '' -p '' --spider HR --regex "."
netexec smb 10.10.10.10 -u '' -p '' --share HR --get-file "Notice from HR.txt" "Notice from HR.txt"

# Recherche les comptes avec SPN
netexec ldap DC_IP -u '' -p '' --query "(servicePrincipalName=*)" "sAMAccountName servicePrincipalName"
```

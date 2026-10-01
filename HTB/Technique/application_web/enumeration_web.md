## Enumération Web Application - Méthodologie complète

---

### Le contexte Layover comme exemple

Tu arrives sur `portal.international.htb` avec des credentials `jenny`. Voici tout ce que tu aurais dû faire systématiquement.

---

## PHASE 1 - Reconnaissance passive (sans être connecté)

### Identifier la technologie
```bash
# Headers HTTP
curl -I http://portal.international.htb
# Cherche : X-Powered-By, Server, Set-Cookie

# Code source de la page
curl -s http://portal.international.htb | grep -i "meta\|generator\|powered\|cms\|framework"
# → <meta name="generator" content="Craft CMS">

# Outil automatique
whatweb http://portal.international.htb
wappalyzer  # extension navigateur

# Fichiers standards
curl http://portal.international.htb/robots.txt
curl http://portal.international.htb/sitemap.xml
curl http://portal.international.htb/.well-known/security.txt
```

### Trouver la version exacte
```bash
# Dans le HTML source
curl -s http://portal.international.htb/admin/login | grep -i "version\|ver="

# Dans les fichiers JS/CSS (souvent versionnés)
curl -s http://portal.international.htb/admin/login | grep -o 'src="[^"]*"'
# → /cpresources/app.js?v=5.9.8

# Changelog public
curl http://portal.international.htb/changelog
curl http://portal.international.htb/CHANGELOG.md

# GitHub du CMS
# https://github.com/craftcms/cms → composer.json → version
```

### Google dorks
```
site:portal.international.htb
site:portal.international.htb filetype:php
site:portal.international.htb inurl:admin
"portal.international.htb" "index of"
```

---

## PHASE 2 - Reconnaissance active (sans être connecté)
```
200 → Page accessible    = Explorer immédiatement
301/302 → Redirection    = Suivre
403 → Forbidden          = Noter, creuser plus tard
404 → Not found          = Ignorer
```
### Dirbusting
```bash
# Wordlist générique
gobuster dir -u http://portal.international.htb \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt,zip,bak \
  -t 50

# Wordlist spécifique CMS
gobuster dir -u http://portal.international.htb \
  -w /usr/share/seclists/Discovery/Web-Content/CMS/craftcms.txt

# Fichiers sensibles
gobuster dir -u http://portal.international.htb \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -x env,bak,old,sql,zip,tar.gz

# Sous-domaines
gobuster vhost -u http://international.htb \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### Fichiers sensibles à tester manuellement
```bash
# Configs exposées
curl http://portal.international.htb/.env
curl http://portal.international.htb/.env.local
curl http://portal.international.htb/.env.backup
curl http://portal.international.htb/config.php
curl http://portal.international.htb/composer.json    # ← révèle les dépendances !
curl http://portal.international.htb/composer.lock    # ← versions exactes !

# Backups
curl http://portal.international.htb/backup.zip
curl http://portal.international.htb/backup.sql
curl http://portal.international.htb/www.zip

# Git exposé
curl http://portal.international.htb/.git/HEAD
# Si accessible → git-dumper pour récupérer tout le code source
git-dumper http://portal.international.htb/.git /tmp/source
```

**`composer.json` est crucial** — il révèle toutes les dépendances et leurs versions sans être connecté :
```json
{
    "require": {
        "craftcms/cms": "^5.9.8",
        "yiisoft/yii2": "2.0.48",
        "psy/psysh": "^0.11"    ← ConsoleProcessus est là !
    }
}
```

---

## PHASE 3 - Analyse après connexion (jenny connectée)

### Explorer l'interface admin
```
Questions à se poser :
□ Quelles fonctionnalités sont accessibles avec ce compte ?
□ Quelles fonctionnalités sont inaccessibles (403) ?
□ Y a-t-il des uploads de fichiers ?
□ Y a-t-il des champs de template/code ?
□ Y a-t-il des imports de données ?
□ Y a-t-il des éditeurs WYSIWYG ?
```

### Intercepter tout le trafic avec Burp Suite
```
1. Configurer Burp comme proxy
2. Naviguer dans TOUTE l'interface admin
3. Regarder chaque requête dans l'historique Burp
4. Chercher :
   - Paramètres JSON complexes
   - Paramètres qui ressemblent à des classes PHP
   - Endpoints qui acceptent des objets sérialisés
   - Paramètres "elementType", "condition", "criteria"
```

### Tester les endpoints API
```bash
# Lister les endpoints connus de CraftCMS
# Google : "CraftCMS API endpoints"
# GitHub : grep -r "registerRoute\|urlManager" craftcms/cms

# Tester manuellement
curl -s -b cookies.txt \
  "http://portal.international.htb/index.php?action=app/get-app-data"

curl -s -b cookies.txt \
  "http://portal.international.htb/index.php?action=users/get-remaining-session-time"

# Endpoints qui acceptent des données POST complexes
curl -s -b cookies.txt -X POST \
  -H "Content-Type: application/json" \
  "http://portal.international.htb/index.php?p=admin&action=element-indexes/get-elements" \
  -d '{"elementType":"craft\\elements\\Entry"}'
```

---

## PHASE 4 - Analyse de la surface d'attaque

### Questions à se poser sur chaque fonctionnalité

#### Sur les formulaires
```
□ Est-ce qu'il y a de l'injection SQL ?
   → ' OR '1'='1
□ Est-ce qu'il y a du XSS ?
   → <script>alert(1)</script>
□ Est-ce qu'il y a du SSTI (Server Side Template Injection) ?
   → {{7*7}} ou ${7*7}
□ Est-ce qu'il y a du path traversal ?
   → ../../../etc/passwd
```

#### Sur les uploads
```
□ Quels formats sont acceptés ?
□ Est-ce que je peux uploader un .php ?
□ Est-ce que le nom de fichier est sanitisé ?
□ Où est stocké le fichier ?
□ Est-ce que je peux accéder au fichier uploadé ?
```

#### Sur les données JSON/sérialisées
```
□ Est-ce que des classes PHP sont référencées ?
   → "class": "...", "elementType": "..."
□ Est-ce que je peux changer le nom de la classe ?
□ Est-ce que je peux passer des arguments au constructeur ?
□ Y a-t-il de la désérialisation PHP ?
   → O:8:"stdClass":0:{}
```

---

## PHASE 5 - Recherche de vulnérabilités spécifiques

### Pour chaque composant identifié
```bash
# CraftCMS 5.9.8
searchsploit craftcms
searchsploit "craft cms"
# Google : "CraftCMS 5.9.8 CVE"
# GitHub : https://github.com/craftcms/cms/security/advisories

# Yii2 (framework)
searchsploit yii2
searchsploit yii
# Google : "Yii2 RCE deserialization"
# Google : "Yii2 behavior injection"

# PHP 8.3
searchsploit php 8.3
# Google : "PHP 8.3 vulnerabilities"

# Nginx (serveur web)
searchsploit nginx
# Google : "nginx alias traversal"
```

### Vérifier chaque dépendance de composer.lock
```bash
# Télécharger composer.lock si accessible
curl http://portal.international.htb/composer.lock -o composer.lock

# Extraire toutes les versions
cat composer.lock | python3 -c "
import json,sys
data = json.load(sys.stdin)
for pkg in data['packages']:
    print(pkg['name'], pkg['version'])
"

# Pour chaque package → chercher des CVE
# psy/psysh 0.11.x → ConsoleProcessus → RCE gadget
# yiisoft/yii2 2.0.48 → behavior injection
```

---

## PHASE 6 - Tester la vulnérabilité Yii2

### Identifier les endpoints qui désérialisent
```bash
# Dans Burp, chercher des requêtes avec des paramètres comme :
# elementType, condition, criteria, fieldLayouts

# Tester si le paramètre "as X" est accepté
curl -s -b cookies.txt -X POST \
  -H "Content-Type: application/json" \
  -H "X-Requested-With: XMLHttpRequest" \
  "http://portal.international.htb/index.php?p=admin/actions/element-search/search" \
  -d '{
    "elementType": "craft\\\\elements\\\\Category",
    "as test": {
        "class": "yii\\\\base\\\\Behavior"
    }
  }'
# Si pas d'erreur sur "as test" → le paramètre est traité par Yii2
```

### Trouver les gadgets disponibles
```bash
# Depuis composer.lock tu sais que psy/psysh est installé
# Google : "Psy ConsoleProcessus RCE"
# → Psy\Readline\Hoa\ConsoleProcessus::__construct($cmd)::start()

# Tester avec une commande bénigne d'abord
CMD = "curl http://TON_IP/test"
# Si tu reçois la requête → RCE confirmé
```

---

## Checklist Web App complète

```
SANS ÊTRE CONNECTÉ
□ curl -I → headers (Server, X-Powered-By, cookies)
□ curl page → meta generator, commentaires HTML
□ whatweb / wappalyzer → stack technique
□ robots.txt, sitemap.xml
□ composer.json, composer.lock → dépendances et versions
□ .env, .git/HEAD → fichiers sensibles exposés
□ gobuster → répertoires et fichiers cachés
□ searchsploit + Google → CVE sur chaque composant

CONNECTÉ (jenny)
□ Explorer toute l'interface → cartographier les fonctionnalités
□ Burp Suite → intercepter toutes les requêtes
□ Identifier les endpoints qui acceptent du JSON complexe
□ Chercher les paramètres "class", "elementType", "condition"
□ Tester SSTI, SQLi, XSS sur chaque champ
□ Tester les uploads
□ Tester les endpoints API documentés et non documentés
□ Vérifier les erreurs → stack traces révèlent la structure

POST-EXPLOITATION (www-data)
□ cat /var/www/portal/.env → credentials, clés
□ composer.json → dépendances installées
□ find /var/www -name "*.php" → code custom
□ mysql → tables custom → données sensibles
□ find / -name "*.conf" → autres services configurés
```

---

## La vraie différence entre un bon et un mauvais CTF player

| Mauvais réflexe | Bon réflexe |
|----------------|-------------|
| Lancer gobuster et attendre | Lire le code source HTML d'abord |
| Chercher des exploits sans identifier la version | Identifier version exacte puis chercher |
| Tester des payloads au hasard | Comprendre la stack avant de tester |
| Ignorer composer.json | Lire toutes les dépendances |
| Passer 2h sur SQLi alors que c'est du Yii2 | Identifier la techno → chercher vulns spécifiques |
| Copier un exploit sans comprendre | Lire l'exploit → adapter → comprendre |

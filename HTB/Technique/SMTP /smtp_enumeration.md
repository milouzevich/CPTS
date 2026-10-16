**Enumération SMTP (port 25)**

**1. Banner grabbing**
```bash
nc -nv IP 25
telnet IP 25
```

**2. Commandes manuelles SMTP**
```bash
EHLO test          # liste les extensions supportées
VRFY root          # vérifie si un user existe (souvent désactivé)
EXPN root          # expand mailing list
RCPT TO:<root>     # teste si user valide
```

**3. Enumération users avec smtp-user-enum**
```bash
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t IP
smtp-user-enum -M RCPT -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t IP
smtp-user-enum -M EXPN -U wordlist.txt -t IP
```

**4. Nmap scripts**
```bash
nmap -p 25 --script smtp-enum-users,smtp-commands,smtp-open-relay IP
```

**5. Open relay test (envoi d'email)**
```bash
EHLO attacker.com
MAIL FROM:<attacker@attacker.com>
RCPT TO:<victim@gmail.com>
DATA
Subject: test
test email
.
QUIT
```

**6. Swaks (outil d'envoi SMTP)**
```bash
swaks --to victim@domain.com --from attacker@domain.com --server IP
swaks --to admin@trick.htb --from test@trick.htb --server IP --body "test" --header "Subject: test"
```

**Ce qu'on cherche :**
- Users valides → bruteforce SSH/autres services
- Open relay → phishing interne
- Version du service → CVE

Sur Trick spécifiquement, SMTP sert à trouver des usernames valides pour ensuite énumérer des vhosts.

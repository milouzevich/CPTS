certipy shadow — Shadow Credentials

C'est une technique complètement séparée d'ADCS. Elle n'exploite pas les templates de certificats — elle exploite l'attribut msDS-KeyCredentialLink directement sur un objet AD.

La mécanique en simple :

Normalement un compte s'authentifie avec son mot de passe. Mais Windows supporte aussi l'authentification par certificat (PKINIT) — et pour ça il regarde l'attribut msDS-KeyCredentialLink de l'objet AD pour savoir quelles clés publiques sont autorisées à s'authentifier comme ce compte.

Si tu as WriteProperty ou GenericWrite sur un compte, tu peux ajouter ta propre clé dans cet attribut — et donc t'authentifier en tant que ce compte via PKINIT sans connaître son mot de passe. Certipy récupère ensuite le hash NT en échange.

Quand l'utiliser :

Situation	Outil
Tu as GenericWrite/WriteProperty sur un compte utilisateur	certipy shadow
Tu as accès à un template de certificat mal configuré	certipy req (ESC1, ESC4...)
Tu as ManageCertificates sur la CA	certipy ca (ESC7)

Les sous-commandes :

bash
# Tout en une fois (ajoute la clé, auth, récupère hash, restore)
certipy shadow auto -u 'USER@domaine.htb' -p 'PASS' -account CIBLE -dc-ip IP

# Juste ajouter la clé (sans auto-restore)
certipy shadow add -u 'USER@domaine.htb' -p 'PASS' -account CIBLE -dc-ip IP

# Lister les clés existantes sur un compte
certipy shadow list -u 'USER@domaine.htb' -p 'PASS' -account CIBLE -dc-ip IP

# Supprimer une clé ajoutée (cleanup manuel)
certipy shadow remove -u 'USER@domaine.htb' -p 'PASS' -account CIBLE -device-id ID -dc-ip IP

Toujours préférer auto — il restaure automatiquement les anciennes clés après avoir récupéré le hash, ce qui évite de casser l'authentification du compte cible.

Résumé en une ligne :

certipy shadow = obtenir le hash NT d'un compte sans changer son mot de passe, en abusant de WriteProperty sur msDS-KeyCredentialLink — nécessite LDAPS.

Ajoute ça dans ton playbook à côté de dacledit et bloodyAD — c'est la technique à sortir dès que tu as GenericWrite sur un compte et que tu ne veux pas (ou ne peux pas) changer son mot de passe.

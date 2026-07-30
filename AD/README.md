## 🧠 Arbre de décision - Sans credentials

```
Nmap
│
├── SMB
│      │
│      ├── Anonymous ?
│      ├── Guest ?
│      └── Shares ?
│
├── LDAP
│      │
│      ├── Anonymous bind ?
│      └── Informations LDAP ?
│
├── RPC
│      │
│      ├── RID Cycling ?
│      └── Enum Users ?
│
├── Kerberos
│      │
│      ├── User Enumeration
│      ├── AS-REP Roast
│      └── Password Spray (si autorisé)
│
└── DNS
       │
       ├── Zone Transfer ?
       ├── SRV Records ?
       └── Nom du domaine ?


```text
==============================
CHECKLIST ACTIVE DIRECTORY
==============================

□ La machine est-elle un DC ?

□ SMB anonyme ?
□ SMB Guest ?

□ LDAP anonyme ?

□ Politique de mot de passe ?

□ Utilisateurs ?

□ Groupes ?

□ Ordinateurs ?

□ Descriptions ?

□ SPN ?

□ Comptes AS-REP Roastables ?

□ Shares SMB ?

□ WinRM accessible ?

□ BloodHound dès qu'un compte est obtenu

□ Quelles ACL ?

□ Quel prochain chemin ?
```

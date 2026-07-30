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


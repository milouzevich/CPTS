
```
              NMAP
                ↓
          Identifier le DC
                ↓
       ┌────────┼─────────┐
       ↓        ↓         ↓
      SMB      LDAP     Kerberos
       ↓        ↓         ↓
   anonymous  bind     users
   guest      users    AS-REP
   shares     groups   SPN
   RID        objects
   users      attrs
       ↓        ↓         ↓
       └────────┼─────────┘
                ↓
         Credentials ?
                ↓
        Authenticated enum
                ↓
          BloodHound
                ↓
         ACL / Groups
                ↓
       Attack Paths
                ↓
       Chemin le plus court
```

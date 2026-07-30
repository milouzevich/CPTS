

                  ACTIVE DIRECTORY
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
    Kerberos             ACL             Réplication
        │                 │                 │
        ↓                 ↓                 ↓
 Kerberoasting       GenericWrite        DCSync
 AS-REP              WriteDACL           │
        │             WriteOwner          ↓
        ↓             GenericAll       secrets
      TGS               │                 │
        │               ↓                 ↓
        ↓          Shadow Creds        NTLM hashes
   crack offline        RBCD                │
                        │                   ↓
                        ↓               PTH / auth
                       S4U

# Questions à se poser à chaque énumération : 
Sans Credential
```

        Quelle identité ?
              ↓
        Quels groupes ?
              ↓
        Quels droits ?
              ↓
        Quels services ?
              ↓
        Quelles relations ?
              ↓
        Quels chemins ?
```


Avec des credential
```
        CREDENTIAL 
              ↓
        VALIDER l'authentification
              ↓
        Quelle identité ?
              ↓
        Quels groupes ?
              ↓
        Quels droits ?
              ↓
        Quels services ?
              ↓
        Quelles relations ?
              ↓
        Quels chemins ?

```
```
                 NOUVEAU CREDENTIAL
                        ↓
                 AUTHENTIFICATION
                        ↓
                   WHO AM I?
                        ↓
             ┌──────────┼──────────┐
             ↓          ↓          ↓
          GROUPS      RIGHTS    SERVICES
             ↓          ↓          ↓
             └──────────┼──────────┘
                        ↓
                   BLOODHOUND
                        ↓
                  ATTACK PATHS
                        ↓
                SHORTEST PATH
```

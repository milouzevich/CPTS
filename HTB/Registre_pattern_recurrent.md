# Vulnérabilité classique

Pattern / technique	| Vu sur	| Contexte de réapparition probable |
--- | --- | ---
Bruteforce login CMS avec wordlist générée via cewl (pas rockyou direct) |	Blunder	| Tout CMS/panel admin avec contenu textuel exploitable


# Partie Privesc 
Pattern / technique	| Vu sur	| Contexte de réapparition probable |
--- | --- | ---
SeMachineAccountPrivilege + Account Operators → penser noPac | Forest | shadow credentials sur les AD avec délégation permissive
ReadGMSAPassword → lire msDS-ManagedPassword → hash NT du compte GMSA |	Search |	Tout AD avec comptes de service GMSA + droits délégués via ACL
Réflexe : droit inhabituel → HackTricks + BloodHound Help → adapter |	Search	| Toutes les machines AD Hard/Insane
GenericAll sur user → Set-ADAccountPassword ou net user	| Search |	Toute chaîne AD avec ACL abuse (Hard/Insane systématiquement)
GenericAll sur group → Add-ADGroupMember	| Search |	Idem — souvent enchaîné avec ReadGMSAPassword ou DCSync

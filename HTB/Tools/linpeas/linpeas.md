Process de lecture LinPEAS — dans cet ordre
```bash 
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
```

1. Groupes du user en premier (avant même de lire le reste)

uid=... groups=...

Les groupes non standards = surface d'attaque directe.

2. Processus inhabituels
Chercher : scripts custom en cours d'exécution, processus qui tournent avec des privilèges élevés, services qui écoutent en localhost.

3. Variables d'environnement
Credentials, tokens, clés API dans /proc/*/environ.

4. Fichiers ajoutés par l'utilisateur
Section Executable files potentially added by user — ce qui n'est pas standard sur le système.

5. SUID/Capabilities
Section capabilities sur les binaires — cap_net_raw, cap_setuid etc.

6. Fichiers cachés intéressants
Section All relevant hidden files.

7. Réseau
Interfaces, ports en écoute locale, forwarding activé.

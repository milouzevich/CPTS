# Préparation HTB — Saison 12

> Rappel structure : une Saison HTB dure 13 semaines, 1 machine seasonale sortie chaque semaine, et seuls les 7 premiers jours comptent pour le classement saisonnier. Le but de ce doc est d'arriver à cette échéance avec une méthode fiable, pas juste plus de connaissances techniques.

---

## 1. Board de pistes (à copier pour CHAQUE machine)

But : ne jamais rester bloqué sans savoir quoi faire ensuite. Dès qu'une piste apparaît (port, endpoint, credential potentiel, version), elle va ici — même si tu ne la testes pas tout de suite.

**Machine :** ___________  **Date :** ___________  **Difficulté :** ___________  **OS :** ___________

| # | Piste identifiée | Source (nmap/gobuster/burp/...) | Testée ? | Résultat |
|---|---|---|---|---|
| 1 | | | ☐ | |
| 2 | | | ☐ | |
| 3 | | | ☐ | |
| 4 | | | ☐ | |
| 5 | | | ☐ | |

**Règle d'usage :** si tu bloques plus de 15-20 min sur une piste → reviens ici et prends la ligne suivante non testée. Le stress vient presque toujours d'un board incomplet, pas d'un manque de compétence.

### Checklist rapide de couverture initiale (avant de creuser)
- ☐ Nmap tous ports fait (pas juste top 1000)
- ☐ Chaque service ouvert a eu au moins un passage d'énum de base
- ☐ Code source HTTP regardé (Ctrl+U, JS, commentaires)
- ☐ Vhosts / sous-domaines testés si HTTP présent
- ☐ Version exacte de chaque service notée (pas juste le nom)

---

## 2. Foothold → Privesc (mémo méthodo)

**Foothold obtenu via :** ___________
- ☐ Shell stabilisé (pty python / rlwrap / socat)
- ☐ user.txt récupéré
- ☐ Premier tour rapide : `id`, `sudo -l`, `whoami`, processus, réseau interne

**Privesc — checklist manuelle (en plus de linpeas/winpeas) :**
- ☐ `sudo -l`
- ☐ Binaires SUID/SGID (`find / -perm -4000 2>/dev/null`)
- ☐ Cron jobs / tâches planifiées
- ☐ Services tournant en root/SYSTEM + leur config
- ☐ Credentials en dur (fichiers config, historique bash, .env)
- ☐ Kernel/OS version → exploit connu ?
- ☐ Capabilities (`getcap -r / 2>/dev/null`)
- ☐ Groupes spéciaux (docker, lxd, disk...)

**root.txt obtenu via :** ___________

---

## 3. Debrief post-machine (5 min, à ne JAMAIS sauter)

**Machine :** ___________  **Réussie seul ? Avec aide ? Writeup ?** ___________

1. Qu'est-ce qui m'a bloqué le plus longtemps ?
2. Combien de temps perdu, et sur quelle piste exactement ?
3. Quelle étape de checklist ai-je sautée ou mal faite ?
4. Technique/pattern à retenir pour la prochaine fois :
5. Un mot sur mon état (stress, précipitation, fatigue) : ___________

### Registre de patterns récurrents (à alimenter au fil des machines)
| Pattern / technique | Vu sur (machine) | Contexte de réapparition probable |
|---|---|---|
| | | |
| | | |

---

## 4. Planning d'entraînement (avant/pendant la Saison 12)

Objectif : combler le manque d'Easy, consolider le Medium, et arriver serein sur les machines seasonales (souvent Easy à Medium en début de saison, plus difficiles en fin de saison).

### Phase 1 — Combler les fondations (2-3 semaines)
- ☐ 8-10 machines **Easy** variées (Linux + Windows), en conditions strictes : chrono, pas de writeup avant 1h de blocage réel
- ☐ Répartition volontaire par technique pour éviter les angles morts :
  - ☐ 2× Web (upload, injection, LFI/RFI)
  - ☐ 2× Active Directory basique (enum, Kerberoasting)
  - ☐ 2× Services mal configurés (SMB, FTP, NFS)
  - ☐ 2× Privesc Linux classique (SUID, cron, PATH hijack)
  - ☐ 2× Privesc Windows classique (services, tokens, registre)
- ☐ Board + debrief remplis à chaque machine, sans exception

### Phase 2 — Monter en Medium (3-4 semaines)
- ☐ 6-8 machines **Medium**, rythme ~2/semaine
- ☐ Pour chacune : autorisation d'un hint/writeup partiel après 1h30-2h de blocage réel, mais tu dois refaire la suite seul
- ☐ Focus particulier sur les chaînes multi-étapes (pivot, chaining de vulns) — c'est souvent ce qui manque en Medium par rapport à l'Easy
- ☐ 1 machine AD Medium minimum (si l'AD est un point faible)

### Phase 3 — Rodage pré-saison (1-2 semaines avant le début)
- ☐ 2-3 machines en conditions "seasonales" : un seul essai, chrono réel, pas d'aide du tout
- ☐ Relire ton registre de patterns récurrents en entier
- ☐ Vérifier ton environnement (VPN, outils à jour, notes prêtes, raccourcis/aliases)

### Pendant la saison (13 semaines)
- ☐ Semaine 1-4 (machines généralement plus accessibles) : viser le solve dans les 48h suivant la sortie, pour garder de la marge en cas de blocage
- ☐ Semaine 5-13 (difficulté qui monte) : garder le réflexe board + pas de fixation sur une seule piste
- ☐ Debrief après CHAQUE machine seasonale, réussie ou non — c'est là que la clairvoyance se construit le plus vite

---

## 5. Anti-stress rapide (à relire si tu bloques en plein challenge)

1. Est-ce que mon board a encore des pistes non testées ? → si oui, change de piste.
2. Ai-je vraiment fait le tour complet (tous ports/services) avant de creuser un seul ?
3. Est-ce que je re-vérifie une version exacte avant de chercher un exploit, ou je suppose ?
4. Pause de 5 min si frustration → revenir avec le board, pas avec la même piste en boucle.

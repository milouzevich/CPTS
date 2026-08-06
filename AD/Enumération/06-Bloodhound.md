## BLOODHOUND
ref : https://www.it-connect.fr/chapitres/bloodhound-pathfinding-rechercher-les-chemins-dattaque/
ref : https://www.it-connect.fr/chapitres/bloodhound-search-recherches-sur-un-noeud/


nous pouvons lister les droits d'administration locale de l'utilisateur ou les sessions ouvertes sur les systèmes du domaine **au moment de la collecte** des données.

**Outbound Object Control** : représentent les droits/actions possibles sur d'autres objets, et ce à nouveau de manière directe ou indirecte (via ses appartenances à des groupes par exemple)

**Inbound Object Control** : permettent de lister tous les _nodes_ qui ont des droits particuliers ou actions sur le nôtre (celui que l'on a sélectionné)


https://www.hackingarticles.in/active-directory-enumeration-bloodhound/

# Méthodologie
Exemple pris sur la machine sauna ( Windows - Domain Controller)

```bash 
# Récupération du fichier Zip pour analyse
bloodhound-python -u '<USER>' -p '<PASS>' -ns <IP_DC> -d <NAME_DC> -c All --zip 
bloodhound-python -u 'fsmith' -p 'Thestrokes23' -ns 10.129.41.79 -d EGOTISTICAL-BANK.LOCAL -c All --zip 
bloodhound-start
```


Pour voir les droits de notre users sur les autres objets vérifier les Outbound Object Control

1. users 
2. analyser les users
3. domain admin 
4. analyser qui est membre du groupe


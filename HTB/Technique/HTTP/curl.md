# **Mémo pour utiliser curl dans certaine situation**

```bash
# Recupérer un contenu en HEX et le convertir en chaine de caractère
curl http://valentine.htb/dev/hype_key | xxd -r -p
```

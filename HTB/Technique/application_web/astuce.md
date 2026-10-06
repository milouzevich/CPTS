Si pb avec un vieux certificat et que la page ne s'affiche pas 

Dans Firefox :

**about:config** → cherche `security.tls.version.min` → mets la valeur à `1`

Ou plus simple, utilise directement curl avec `-k` et `--tls-max` :

```bash
curl -k --tls-max 1.2 https://10.129.229.183/
```

Ou pour vraiment forcer le vieux TLS :

```bash
curl -k --ssl-no-revoke --tls-max 1.0 https://10.129.229.183/
```

Qu'est-ce que tu obtiens ?

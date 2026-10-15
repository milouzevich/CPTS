# CPTS Playbook — Offensive Security Notes

> Personal knowledge base built while preparing for the **HTB Certified Penetration Testing Specialist (CPTS)** certification and practicing on Hack The Box labs. Covers Active Directory attack chains, web application exploitation, and a curated set of techniques and tools encountered across real engagements and CTF-style environments.

---

## Contents

### 🏴 Active Directory

Structured methodology for attacking Windows enterprise environments — from initial enumeration to full domain compromise.

| Section | Topics |
|---|---|
| [Checklist](AD/01-Ckecklist_AD.md) | Pre-engagement checklist for AD assessments |
| [Methodology](AD/02-%20Méthodologie.md) | Step-by-step attack path from unauthenticated to DA |
| **Enumeration** | |
| ↳ [SMB](AD/Enumération/02-smb_enumeration.md) | Shares, sessions, null sessions |
| ↳ [LDAP](AD/Enumération/03-ldap_enum.md) | Domain objects, users, groups, trusts |
| ↳ [Kerberos](AD/Enumération/04-Kerberos_enum.md) | SPN enumeration, AS-REP roasting candidates |
| ↳ [Kerbrute](AD/Enumération/05-kerbrute.md) | Username enumeration via Kerberos pre-auth |
| ↳ [BloodHound](AD/Enumération/06-Bloodhound.md) | Attack path analysis, SharpHound collection |
| **Privilege Escalation via ACL Abuse** | |
| ↳ [Methodology](AD/privesc/00-methodologie.md) | How to read and exploit misconfigured ACLs |
| ↳ GenericAll / GenericWrite | Full control and targeted attribute writes |
| ↳ ForceChangePassword | Password reset without knowing current creds |
| ↳ WriteOwner / WriteSPN | Ownership takeover, Kerberoasting setup |
| ↳ DCSync | Credential extraction via directory replication |
| ↳ AddSelf | Self-addition to privileged groups |
| ↳ AllowedToDelegateTo | Constrained delegation abuse |
| ↳ ReadGMSAPassword / ReadLAPSPassword | Managed account credential retrieval |
| ↳ SeBackupPrivilege / SeEnableDelegation | Token privilege abuse |
| ↳ [Backup Operator](AD/privesc/GROUP-BAckup%20Operator.md) | Group-based local admin escalation |
| ↳ [Server Operator](AD/privesc/GROUP-Server%20Operator.md) | Service control abuse for SYSTEM |
| **ADCS Attacks** | |
| ↳ [Overview](AD/Les%20Attaques%20AD%20/ADCS/ADCS.md) | Certificate Services attack surface |
| ↳ [ESC9](AD/Les%20Attaques%20AD%20/ADCS/ADCS-ESC9.md) | No-security-extension template abuse |
| ↳ [Certipy](AD/Les%20Attaques%20AD%20/ADCS/Certipy.md) | Certificate enumeration and exploitation |
| ↳ [Shadow Credentials](AD/Les%20Attaques%20AD%20/ADCS/certipy_shadow.md) | msDS-KeyCredentialLink manipulation |
| ↳ [Certificate Handling](AD/Les%20Attaques%20AD%20/ADCS/Certificat_pfx_p12.md) | PFX/P12 extraction and reuse |
| **MSSQL** | Linked server exploitation, xp_cmdshell, privilege escalation via SQL |
| **Impacket** | [Command reference](AD/suite_impacket/resume_command.md) — secretsdump, psexec, wmiexec, getTGT, getST |

---

### 🌐 HTB Techniques & Tools

Technique-focused notes organized by attack category. Built from Hack The Box machines and CPTS lab modules.

**Web Application**
- [Web enumeration](HTB/Technique/application_web/enumeration_web.md) — directory busting, parameter discovery, tech fingerprinting  
- [Web analysis methodology](HTB/Technique/application_web/analyse_web.md)
- [Application checklist](HTB/Technique/application_web/checklist.md)
- [LFI techniques](HTB/Technique/LFI/LFI_technique.md) — path traversal, filter chains, log poisoning  
- [PHP exploitation](HTB/Technique/application_web/PHP/php_exploit.md)
- [API testing](HTB/Technique/API/api.md) — method enumeration, authentication bypass, object manipulation

**Infrastructure & Post-Exploitation**
- [System enumeration — Linux](HTB/Technique/Enumeration_system/enum_linux.md)
- [System enumeration — Windows](HTB/Technique/Enumeration_system/enum_windows.md)
- [Linux privilege escalation](HTB/Technique/Privesc/linux.md)
- [Reverse shells](HTB/Technique/reverse_shell/reverse_shell.md) — bash, Python, PHP, socat, mkfifo
- [SSH key techniques](HTB/Technique/cle_shh/ssh.md)
-Création d'un PLAYBOOK, pour les concepts et commandes

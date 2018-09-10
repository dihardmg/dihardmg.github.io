---
layout: post
tags: [Hadoop, Hortonwork Sanbox,remote ssh]
title: "Warning: Remote Host Identification Has Changed"
card-img: 
acknowledgements: "null"
---


Ketika sudah selesai, jalanin sanbox hadoop, saya ingin menajalankan command line di terminal di app, bukan terminal di VM virtualbox karena tidak bisa copy dan paste. Ada kendala ketika remote terminal di `ssh root@127.0.0.1 -p 2222` seperti di bawah ini.

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:EzFsbb6nJ9/JKM2o+jOoKL10x1b6uA9subm8xGBfOoE.
Please contact your system administrator.
Add correct host key in /Users/macbookpro/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/macbookpro/.ssh/known_hosts:4
RSA host key for [127.0.0.1]:2222 has changed and you have requested strict checking.
Host key verification failed.

[Process completed]
```
![ssh remote error](/img/post/001/ssh-error.png)



Untuk masalah di atas, caranya buka terminal, ketikan perintah di bawah ini, dan buka lagi, ketika ada notifikasi seperti ini, ketik `yes`

``
 $ ssh-keygen -R [127.0.0.1]:2222
``
![ssh remote error](/img/post/001/ssh.png)

Setalah ini selesai coba lagi buka remote `ssh root@127.0.0.1 -p 2222` dan hasil nya
![ssh remote error](/img/post/001/sshok.png)

## Penutup
Sampai disini dahulu error karena RSA key fingerprint, sehingga perlu di generate ulang key nya lagi.
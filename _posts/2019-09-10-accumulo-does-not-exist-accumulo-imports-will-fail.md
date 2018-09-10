---
layout: post
tags: [Hadoop, Hortonwork Sanbox]
title: "Eaccumulo does not exist! Accumulo imports will fail"
card-img: 
acknowledgements: "null"
---


Ketika memulai belajar hadoop, dan download file dari <a href="https://hortonworks.com/products/sandbox/#install">hortonwork sandbox</a>, sudah berjalan ada masalah sedikit pada saat cek `$sqoop version` ternyata ada masalah seperti gambar di bawah ini.

```
sqoop version
Warning: /usr/hdp/2.3.2.0-2950/accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
18/09/10 09:28:04 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6.2.3.2.0-2950
Sqoop 1.4.6.2.3.2.0-2950
git commit id 57d698f3586697bb31bad5ee100705ca4d469360
Compiled by jenkins on Wed Sep 30 20:33:34 UTC 2015
```
![accumulo does not exist! Accumulo imports will fail](/img/post/001/sqoop-error.png)

Untuk masalah di atas ikuti penyelesaian nya seperti ikuti baris command line di bawah ini di sanbox handoop VM


```
$ sudo mkdir /var/lib/accumulo
$ ACCUMULO_HOME='/var/lib/accumulo'
$ export ACCUMULO_HOME
```
Ini akan men setting `$ACCUMULO_HOME` Kemudian hasilnya, 
![sqoop version](/img/post/001/sqoop.png)

## Penutup
Ok sampai disini dahulu installasi hadoop untuk masalah sqoop tidak berjalan. Dan sekarang it works.
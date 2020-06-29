---
layout: post
tags: [Setting java home, setting maven, MacOS Catalina]
title: "Setting Java Home dan Setting Maven MacOS Catalina"
card-img: 
acknowledgements: "null"
---

# Setting Java home & Install maven di MacOS Catalina

## 1. Overview

Artikel ini akan membahas setting java home dan maven di MacOS Catalina, sebelumnya download maven projek terlebih dahulu di web berikut https://maven.apache.org/download.cgi
selanjutnya ekstrak file dan taruh file maven ke path :
 ```
 /opt/apache-maven
 ```

## 2. Bikin file bernama bash_profile
    $ touch  ~/.bash_profile
## 3. Buka file bash_profile dan ketik berikut    
    //JAVA_HOME
    export JAVA_HOME=$(/usr/libexec/java_home)

    //MAVEN
    export PATH=$PATH:/opt/apache-maven/bin
## 4. Jikalau sudah, dicek apakah sudah berhasil atau belum proses path java home dan maven nya.
    $ echo $JAVA_HOME
    $ mvn -version

![Setting java home dan install maven di mac Os Catalina](/img/post/001/setjavahomedansetmaven.png)
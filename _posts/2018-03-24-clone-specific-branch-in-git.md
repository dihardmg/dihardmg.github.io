---
layout: post
tags: [git, branch, clone]
title: "Clone a specific branch in git"
card-img: img/post/001/1803241.png
acknowledgements: "Null"
---

git clone pada brach tertentu pada suatu repo github

## Permasalaah untuk pull pada specific branch:

pada permasalaah ini, saya kebetulan ikut projek baru, tetapi saya harus ngikuti branch 1.0.1 pada saat itu saya bingung bagaimana untuk pull pada branch tertentu. biasanya dulu saya tinggal
```
git Clone git@github.com:user/myproject.git
```
tetapi ini pull pada semuanya, bukan untuk branch dengan specific.

___


## Penyelesaian untuk pull pada specific branch!

penggunaan pull pada branch tertentu sebagai berikut.

 ```
 git clone -b <branch> <remote_repo>
 ```

Sebagai contoh saya akan pull ke branch 1.0.1 pada Repository.
 ```
 git clone -b 1.0.1 git@github.com:user/myproject.git
 ```

Setelah clone pada branch 1.0.1 kita bisa verifikasi benar atau tidak yang kita ambil pada branch 1.0.1
```
git branch
* 1.0.1
```

___

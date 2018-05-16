---
layout: post
tags: [Hibernate, Indexes, Auto index, spring boot]
title: "Belajar hibernate Create Auto Index"
card-img: img/post/001/database-index.jpg
acknowledgements: "null"
---
Kita pernah mendengar tentang index atau indexing, mungkin saja belum atau juga sudah, pada artikel kali ini akan membahas apa itu Indexing database, bagaimana menerapkanya pada database PostgreSQL ke Java Spring boot. 

## Apa itu database index ?
Databasae index adalah struktur data untuk meningkatkan kecepatan operasi pengambilan data pada tabel basis data. Mudahnya menyimpan nilai spesifik sebuah kolom pada sebuah table.

## Apa fungsi database index ?
Indeks digunakan untuk menemukan data dengan cepat tanpa harus mencari setiap baris dalam tabel basis data setiap kali tabel basis data diakses. 
![database indexing](/img/post/001/database-index.jpg)
<small><a href="https://techcrunch.com/2018/03/09/infosums-first-product-touts-decentralized-big-data-insights/">sumber gambar</a></small>

## Contoh table 
Disini saya akan membuat sebuah table dengan nama karyawan untuk describe nya seperti di bawah ini.

```
Table "public.karyawan"
   Column   |          Type          | Collation | Nullable | Default | Storage  | Stats target | Description 
------------+------------------------+-----------+----------+---------+----------+--------------+-------------
 id         | character varying(255) |           | not null |         | extended |              | 
 nama       | character varying(255) |           | not null |         | extended |              | 
 keterangan | character varying(255) |           |          |         | extended |              | 
Indexes:
    "karyawan_pkey" PRIMARY KEY, btree (id)
```

Sedangkan untuk query sql nya sebagi berikut.
```
CREATE TABLE karyawan(
  id character varying(255) NOT NULL PRIMARY KEY,
  nama character varying(255) NOT NULL,
  keterangan character varying(255)
);
````

## Membuat auto index pada spring boot
Sebenernya sangat mudah sekali menambahkan index di java spring boot. Pada table entity menambahkan annotation `@Table` selanjutnya isi dengan nilai indexes, kemudian column yang jadikan index adalah nama `(nama_karyawan_idx)` & keterangan `(ket_karyawan_idx)` ahh ribet amat ya langsung aja deh ke java nya :D

``` java
@Entity
@Table(name = "karyawan",
        indexes = {
                @Index(columnList = "nama", name = "nama_karyawan_idx"),
                @Index(columnList = "keterangan", name = "ket_karyawan_idx")
        })
public class Karyawan {

    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String id;

    @NotEmpty
    @NotNull
    @Size(min = 3, max = 255)
    @Column(nullable = false)
    private String nama;

    @NotEmpty
    @Size(max = 255)
    @Column(nullable = false)
    private String keterangan;
}
```
Sekarang hasil query databasenya  adalah:
```
                                           Table "public.karyawan"
   Column   |          Type          | Collation | Nullable | Default | Storage  | Stats target | Description 
------------+------------------------+-----------+----------+---------+----------+--------------+-------------
 id         | character varying(255) |           | not null |         | extended |              | 
 nama       | character varying(255) |           | not null |         | extended |              | 
 keterangan | character varying(255) |           |          |         | extended |              | 
Indexes:
    "karyawan_pkey" PRIMARY KEY, btree (id)
    "ket_karyawan_idx" btree (keterangan)
    "nama_karyawan_idx" btree (nama)
```


## Penutup
Pada tahap terakhir ini, saya tidak ada unit testing indexing nya, apakah lebih cepat atau lambat pada operasi pencarian data. Sekian #cherrrsss ;)

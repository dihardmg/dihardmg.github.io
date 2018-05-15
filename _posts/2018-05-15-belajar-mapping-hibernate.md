---
layout: post
tags: [mapping, relasi, hibernate, OneToMany, ManyToOne]
title: "Belajar mapping relasi hibernate"
card-img: img/post/001/belajar-mapping-relasi-hibernate.png
acknowledgements: "om Endy "
---
pada dua hari yang lalu saya belajar ngulik mapping relasi di dibernate, dan membuat saya jadi penasaran karena saya belum dong juga ;( untuk menerapkan pada projek kecil kecilan. Setalah berselancar di google, saya menemukan blog yang sangat keren  yaitu punya om endy <a href="https://software.endy.muhardin.com/java/memahami-mapping-relasi-hibernate/">disini</a> dari situ saya mulai paham apa itu Aggregation vs Composition.

## Contoh mapping:

Disini saya akan memulai untuk relasi antara dua table yaitu table karyawan dan alamat. Dimana nanti goal mampping nya pada sisi karyawan ketika di hapus akan juga hapus data child relasi pada table alamat. untuk mapping pada sisi karyawan `@OneToMany` dan di sisi alamat `@ManyToOne`

![Belajar mapping relasi hibernate](/img/post/001/belajar-mapping-relasi-hibernate.png)


```
CREATE TABLE karyawan(
  id character varying(255) NOT NULL PRIMARY KEY,
  nama character varying(255) NOT NULL,
  keterangan character varying(255)
);

CREATE TABLE alamat(
  id character varying(255) NOT NULL PRIMARY key,
  nama character varying(255) NOT NULL,
  alamat character varying(255) NOT NULL,
  id_karyawan character varying(255) NOT NULL
);

ALTER TABLE alamat
ADD CONSTRAINT fk_alamat_karyawan FOREIGN KEY (id_karyawan) REFERENCES karyawan(id);

```


## Hibernate Mapping
Untuk relasi hibernate mapping pada java seperti di bawah ini.

```java
//karyawan
@Data
@ToString(exclude = "listAlamat")
@Entity
@Table(name = "karyawan")
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

    @OneToMany(mappedBy = "karyawan",
            cascade = CascadeType.REFRESH,
            orphanRemoval = true)
    private List<Alamat> listAlamat = new ArrayList<>();
}


//Alamat

@Data
@Entity
@Table(name = "alamat")
public class Alamat {
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String id;

    @NotNull
    @NotEmpty
    @Size(min = 3, max = 150)
    @Column(nullable = false)
    private String nama;

    @Column(nullable = false)
    @NotNull
    @NotEmpty
    private String alamat;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "id_karyawan")
    private Karyawan karyawan;

}
```
Berikut SQL yang dihasilkan untuk hapus data

```
Hibernate: 
    select
        karyawan0_.id as id1_1_0_,
        karyawan0_.keterangan as keterang2_1_0_,
        karyawan0_.nama as nama3_1_0_ 
    from
        karyawan karyawan0_ 
    where
        karyawan0_.id=?
Hibernate: 
    select
        listalamat0_.id_karyawan as id_karya4_0_0_,
        listalamat0_.id as id1_0_0_,
        listalamat0_.id as id1_0_1_,
        listalamat0_.alamat as alamat2_0_1_,
        listalamat0_.id_karyawan as id_karya4_0_1_,
        listalamat0_.nama as nama3_0_1_ 
    from
        alamat listalamat0_ 
    where
        listalamat0_.id_karyawan=?
Hibernate: 
    delete 
    from
        alamat 
    where
        id=?
Hibernate: 
    delete 
    from
        karyawan 
    where
        id=?
```

## Mapping relasi composition
ketika saya menghapus di sisi `One`, maka di sisi `Many` juga ikut terhapus

`@OneToMany(
    mappedBy = "karyawan",
    cascade = CascadeType.REFRESH,orphanRemoval = true)`

- `cascade = CascadeType.REFRESH` supaya untuk refresh ketika data di sisi one di update, maka data di sisi `Many` juga ke update.
- `orphanRemoval = true` supaya ketika data di sisi `One` di hapus, maka di sisi many juga terhapus.

## Penutup
Demikan cerita singkat kali ini mengenai relasi mapping hibernate, semoga berguna,terutama bagi saya sendiri karena saya sering lupa jadi saya menuliskan ini di blog. ;)

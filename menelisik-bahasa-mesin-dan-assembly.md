---
layout: page
title: Menelisik assembly Pt. 1 - kompilasi terbalik
permalink: /menelisik-intuisi-assembly-dan-bahasa-mesin-cpu-reverse-compilation
---
{% include analytics.html %}

# Menelisik intuisi assembly dan bahasa mesin CPU, reverse compilation

> #### [2024-07-15@alfiankan](https://github.com/alfiankan)


![Cpu 100% Fine](/cpu_fine.jpg)

Image by [errantscience.com](https://errantscience.com/){: .text-center}

<br>

>Tulisan ini adalah murni catatan explorasi penulis (Software Engineer) yang lebih fokus ke high level yang belum berpengalaman dalam low level system.
tulisan ini harapanya bisa membantu yang sama sama lagi belajar atau mau memahami dan masuk ke skena ini.


CPU adalah processor tapi processor belum tentu CPU, GPU juga termasuk processor. Namun tulisan ini akan membahas CPU, bagaimana dia bisa memahami dan bekerja sesuai apa yang kita mau.
{: .text-justify}

## Memahami CPU sebagai kotak hitam ajaib

Oke secara abstrak dan dasar CPU akan melakukan pemrosesan aritmatika logika dan juga pertukaran data input/output semuanya secara primitif. Primitif disini maksudnya bahkan
tidak ada operasi matematika tertentu pada sebagian arsitektur CPU seperti pembagian, namun operasi pembagian ini dapat dicapai dengan beberapa kombinasi intruksi, 
ibaratkan hidup di zaman bantu anda akan diberi basic tool untuk meneruskan hidup, anda dapat membuat apapun dari itu. 
Tulisan ini tidak akan sedalam menelisik sampai ke logic gate, komponen penting dalam CPU namun akan di level bagaimana komponenen di dalam CPU melakukanya.

Dalam artikel ini akan kita pilih dan bahas satu CPU saja yaitu arsitektur ARM dengan 32 bit, kenapa tidak 64? pada artikel lainya dan kedepan nya mungkin akan menggunakan 64 bit
namun dalam artikel ini cukup menggunakan 32 bit lebih simpel untuk ditampilkan dalam ilustrasi gambar di artikel ini. Kenapa ARM? karena berbasis RISC yang lebih sederhana.
Coba kita lihat gambaran dalam CPU di bawah.
{: .text-justify}

![Cortex M3](/cortex-m3.jpg)

Untuk artikel ini kita akan banyak membahas tiga komponen yang di tandai merah saja, karena sangat banyak komponen yang komplek, kita belum akan terjun sedalam itu, karena basic
saya adalah software engineer yang hari harinya lebih banyak bikin REST API dan segala backend system, maka hal yang pingin penulis tau terlebih dahulu adalah
sebenarnya bagaimana kode yang penulis tulis bisa di eksekusi oleh CPU? dan memang CPU tidak akan paham code berikut.

```c
int main() {
    int a = 9;
    int b = 10;
    int c = a + b;
}
```

Oke dari gambar diatas ada 3 komponen yang akan banyak terkait pada pembahasan artikel ini yaitu:

- ALU: Aritmethic and Logic unit, dari namanya sudah menjelaskan di komponen ini dapat menerima input, melakukan operasi matematika dan logika lalu memberikan outpur
- Decoder: disini instruksi yang kita buat dalam binary akan di terjemahkan untuk di tentukan eksekusi apa selanjutnya.
- Instruction Fetch Unit: disini tugasnya mengambil instruction dari Memmory, kode kita yang di load ke memori akan dibaca secara runut jika kita dalam 32 bit maka akan menggunakan 32 bit panjangnya.

Oke penulis akan coba memberikan gambaran paling abstrak proses dari intruksi ke process dalam CPU. Ilustrai ini bukan ilustrasi lengkap namun hanya bagaimana
perintah atau intruksi yang kita mau sampai ke ALU untuk melakukan tugas matematis yaitu menambahkan 2 angka.

```bash
|addr|MEMORY======|
|---------------------|
|0   | SIMPAN KE r2, 3|---------[FETCH]--------[DECODE]------[ALU]
|---------------------|            ↓
|1   | SIMPAN KE r3, 5|            ↓
|---------------------|            ↓
|2   | TAMBAH r2, r3  |            ↓
|---------------------|            ↓ 
|3   | NOP            |         Arah Fetch
|---------------------|

NOP = Tidak melakukan apa apa

```

## Dari bahasa kita ke bahasa mesin

CPU dibuat berdasarkan kemajuan matematika dalam interpretasi bilangan biner dan perkembangan sophistikatik oleh para ilmuan hebat. Jadi CPU hanya bisa paham binary data representasi 0 dan 1 yang kombinasinya dapat di translate ke representasi lain hexa octal decimal dll. 
Kali ini kita akan coba melakukan reverse compilation dari bahasa mesin ke bahasa high level, kita coba membalik cara memahami dari kode mesin ke level lebih tinggi.

Contoh kita punya sebuah program sudah dalam bentuk binary yang mana sudah siap di jalankan oleh CPU, program ini sangat sederhana. Yaitu menjumlahkan 2 angka saja lalu menyimpan hasilnya ke suatu variable.

```bash
000001010001000010100000111000110000001100100000101000001110001100000010001100001000000111100000
```

Suka atau tidak suka CPU hanya akan paham perintah dalam format seperti diatas? kalau tidak ada inovasi Assembly, dan para orang orang hebat yang membuat bahasa pemograman
mungkin sekarang untuk memprogram kita perlu menulis literal seperti itu 0 dan 1, mungkin keyboard kita cuman akan begini.

![Binary Keyboard](/binary_keyboard.jpg)


Kita akan coba jalankan dengan qemu arm 32 bit, kenapa dengan qemu? karena kita ingin melihat program kita berjalan standalone di mesin (baremetal) bukan dengan pengaruh sistem operasi,
jika teman teman ingin menginstall untuk mencobanya bisa cek dokumentasi berikut [Download Qemu](https://www.qemu.org/download/).

```bash
qemu-system-arm -M vexpress-a9 -m 32M -no-reboot -nographic -kernel rom.bin
```
- `-M vexpress-a9` kita set mengunakan mesin ini
- `-m 32` kita set pakai RAM 32MB saja
- `-no-reboot -nographic` kita tidak mau ada tampilan dan tidak ingin restart kalau program error
- `-kernel` kita set path file program kita dalam binary

Sebenarnya banyak tool untuk melakukan diasembly untuk mengubah binary menjadi assembly yang lebih mudah dibaca, kita coba dulu untuk diassembly dengan tool bawaan gnuc compiler
hasilnya nanti kita jadikan pedoman bahwa program ini harunsya berjalan demikian. dan kita jadikan referensi untuk kita coba melakukan diassembly secara manual.

```bash
> arm-none-eabi-objdump -m ARM -b binary -D rom.bin

rom.bin:     file format binary


Disassembly of section .data:

00000000 <.data>:
   0:	e3a01005 	mov	r1, #5
   4:	e3a02003 	mov	r2, #3
   8:	e0813002 	add	r3, r1, r2
```

dalam struktur intruksi mesin yang akan di baca cpu akan dibaca per baris instruksi se ukuran bit nya, misalkan yang kita pakai sekarang adalah 32 bit maka akan mengambil
sebanyak 32 bit maksimal sebagai satu baris (ada mode thumb, mmungkin dapat dicari sendiri, singkatnya kita dapat menggunakan lebih sedikit bit untuk sebuah intruksi).

Sekarang kita coba melakukan diasembly secara manual untuk membuktukan bahwa program kita dalam binary sebelumnya sesuai dengan hasil diassembly oleh tool diatas. untuk
melakukanya biasanya setiap CPU akan punya dokumentasi yang sangat lengkap termasuk yang kita gunakan saat ini, untuk dokumentasi yang akan kita gunakan sekarang anda
dapat mendownload dari sini [Download](https://developer.arm.com/documentation/ddi0487/latest/)

Pertama kita coba pisahkan kode binary kita tadi per 32 bit, kita akan anggap itu adalah satu baris instruksi. per basinya kita pecah lagi per 1 byte (8 bit)

```bash
00000101 00010000 10100000 
11100011 00000011 00100000
10100000 11100011 00000010 
00110000 10000001 11100000
```

Mari kita mulai, kita mulai baca dari instruksi pertama, sebelumnya kita harus memahami bahwa pada kata pertama dalam assembly disebut opcode, opcode sebenarnya beruba
binary juga namun untuk mempermudah dalam pe referensian dokumentasi maka digunakan mnemonic contohnya MOV. jika kita ambil semuanya dari program kita maka akan ada bebera
opcode dan atau mnemonic.

- MOV
- ADD

Sebelum melanjutkan ke proses reverse compiling kita harus mengetahui bahwa di dalam CPU juga terdapat memori yang paling kencang dan paling dekat dengan processing unit,
ini bisa di ibaratkan seperti local variabel ketika kita menggunakan bahasa pemograman, namun variabel ini terbatas, bisa kita manfaatkan untuk berbagai hal termasuk manipulasi data
memori akses, dan beberapa register juga dibuat khusus untuk keperluan terntentu. pada artikel ini kita akan cukup mengetahui General purpose register kita bisa gunakan untuk
apa saja ini berjulah terbata, diawali dengan prefix `r<bumber>` contohnya r0, r1, r2, r3 sampai 15, lalu ada register register khusus namun pada artikel ini kita cukup
mengetahui satu dulu yaitu PC atau program counter, gunananya untuk menyimpan penunjuk (pointer) address pada intruksi mana program yang akan di fetch.

```bash
R00=00000000 R01=00000005 R02=00000003 R03=00000008
R04=00000000 R05=00000000 R06=00000000 R07=00000000
R08=00000000 R09=00000000 R10=00000000 R11=00000000
R12=00000000 R13=00000000 R14=00000000 R15=60010010
PC=00000000
```

### Baris pertama `mov r1, #5`

Kita akan coba cari MOV dari dokumentasi ARM dan kita akan mendapat berikut.

![MOV](/MOV.png)

Oke kita coba lihat pada dokumentasi diatas sudah di gambarkan detail bit bit apa yang perlu kita tulis dari bit 0 sampai 31 (32 bit)
kita coba buat placeholdernya terlebih dahulu.

```bash

31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0
?  ?  ?  ?  0  0  1  1  1  0  1  ?  0  0  0  0  ?  ?  ?  ?  ?  ?  ? ? ? ? ? ? ? ? ? ?

```

pada bit ke 28 - 31 adalah cond, untuk mengisi bit ini kita bisa lihat tabel berikut. untuk operasi tanpa kondisi tertentu
kita cukup pakai `1110`

![Condition](/cond.png)

Kita coba masukan bit kondisi ke template kita sebelumnya.
    
```bash

31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0
1  1  1  0  0  0  1  1  1  0  1  ?  0  0  0  0  ?  ?  ?  ?  ?  ?  ? ? ? ? ? ? ? ? ? ?

```

lalu untuk bit ke 20 adalah flag `S` kita belum akan membahas dan menggunakan bit ini maka kita set ke 0
  
```bash

31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0
1  1  1  0  0  0  1  1  1  0  1  0  0  0  0  0  ?  ?  ?  ?  ?  ?  ? ? ? ? ? ? ? ? ? ?

```

Selanjutnya adalah pada bit ke 12 sampai 15 kita bisa mengisikan nomor `register` yang akan kita gunakan untuk menyimpan immidiete value (value langsung), jika kita
tilik ulang intruksi nya `mov r1, #5` kita dapat mengartikan memasukan angka 5 (# artinya immidiete) ke register `r1`, untuk mengisi kita cukup masukan 1 dalam binary
dalam ukuran 4 bit `0001`,

```bash

31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0
1  1  1  0  0  0  1  1  1  0  1  0  0  0  0  0  0  0  0  1  ?  ?  ? ? ? ? ? ? ? ? ? ?

```

lalu yang terakhir adalah immidiete value, kita mengisikan bit dari angka 5 sepanjang 12 bit yaitu `000000000010`


```bash

31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0
1  1  1  0  0  0  1  1  1  0  1  0  0  0  0  0  0  0  0  1  0  0  0 0 0 0 0 0 0 1 0 1

```

Oke semua sudah terisi mari kita coba cocokan untuk cek apakah hasilnya benar dengan `00000101 00010000 10100000 11100011`, namun kita perlu
mengurutkan ulang karena pada arsitektur yang kita pakai ini adalah little endian dimana LSB (Least Significant Bit) disimpan di paling kiri, ata kita bisa
membacanya dari kanan ke kiri

```bash
hasil decode manual             = 11100011 10100000 00010000 00000101
kode program                    = 00000101 00010000 10100000 11100011
kode program dibaca dari kanan  = 11100011 10100000 00010000 00000101

komparasi: MATCH
```
Untuk dua instruksi selanjutnya caranya sama yaitu cek dokumentasi nya untuk melihat urutan meletakan bit dan ikuti instruksinya.
intruksi pertama ini akan menyimpan angka `5` ke register `r1`,

### Baris kedua `mov r2, #3`

Pada baris kedua ini mirip dengan bars pertama hanya beda value saja, jika ingin mencoba melakukan decode manual seperti baris pertama anda dapat mengikuti flow sebelumnya.
pada baris kedua akan menyimpan angka `3` ke register `r2`,


### Baris ketiga `add r3, r1, r2`

Pada baris ketiga adalah `ADD` jika dilihat dari dokumentasi urutan peletakan bit adalah seperti dibawah,

![ADD](/ADD.png)

Jika kita coba lakukan proses yang sama seperti pada bariis pertama, kita akan mendapatkan bahwa hasil diassembly cocok dengan kode binary dalam program kita.
bedanya pada opcode `ADD` (mnemonic) ini membutuhkan 3 register.
jika kita lihat pada gambar ada,

- Rn: kita bisa letakan register 1 disini
- Rm: kita bisa letakan register 2 disini
- Rd: kita letakan register untuk menyimpan hasil penjulahan yaitu `r3`


Sampai disini kita telah menyampai titk terang program kita tadi sudah benar sesuai hasil diassembly dan kita bisa memahami programnya yaitu sebagai berikut.

```bash

mov r1, #5          // simpan angka 5 ke register r1
mov r2, #3          // simpan angka 3 ke register r2
add r3, r1, r2      // tambahkan register r1 + r2 lalu simpan hasilnya di register r3

```

Sekarang kita akan coba jalankan program kita tadi ke `qemu` dan melihat hasilnya dengan `qemu monitor`

```bash
> qemu-system-arm -M vexpress-a9 -m 32M -no-reboot -nographic  -kernel rom.bin
> qemu-system-arm -M vexpress-a9 -m 32M -no-reboot -nographic  -kernel rom.bin
QEMU 9.0.1 monitor - type 'help' for more information
(qemu) info registers

CPU#0
R00=00000000 R01=00000005 R02=00000003 R03=00000008
R04=00000000 R05=00000000 R06=00000000 R07=00000000
R08=00000000 R09=00000000 R10=00000000 R11=00000000
R12=00000000 R13=00000000 R14=00000000 R15=64add6d4
PSR=400001d3 -Z-- A S svc32
s00=00000000 s01=00000000 d00=0000000000000000
s02=00000000 s03=00000000 d01=0000000000000000
s04=00000000 s05=00000000 d02=0000000000000000
s06=00000000 s07=00000000 d03=0000000000000000
s08=00000000 s09=00000000 d04=0000000000000000
s10=00000000 s11=00000000 d05=0000000000000000
s12=00000000 s13=00000000 d06=0000000000000000
s14=00000000 s15=00000000 d07=0000000000000000
s16=00000000 s17=00000000 d08=0000000000000000
s18=00000000 s19=00000000 d09=0000000000000000
s20=00000000 s21=00000000 d10=0000000000000000
s22=00000000 s23=00000000 d11=0000000000000000
s24=00000000 s25=00000000 d12=0000000000000000
s26=00000000 s27=00000000 d13=0000000000000000
s28=00000000 s29=00000000 d14=0000000000000000
s30=00000000 s31=00000000 d15=0000000000000000
s32=00000000 s33=00000000 d16=0000000000000000
s34=00000000 s35=00000000 d17=0000000000000000
s36=00000000 s37=00000000 d18=0000000000000000
s38=00000000 s39=00000000 d19=0000000000000000
s40=00000000 s41=00000000 d20=0000000000000000
s42=00000000 s43=00000000 d21=0000000000000000
s44=00000000 s45=00000000 d22=0000000000000000
s46=00000000 s47=00000000 d23=0000000000000000
s48=00000000 s49=00000000 d24=0000000000000000
s50=00000000 s51=00000000 d25=0000000000000000
s52=00000000 s53=00000000 d26=0000000000000000
s54=00000000 s55=00000000 d27=0000000000000000
s56=00000000 s57=00000000 d28=0000000000000000
s58=00000000 s59=00000000 d29=0000000000000000
s60=00000000 s61=00000000 d30=0000000000000000
s62=00000000 s63=00000000 d31=0000000000000000
FPSCR: 00000000
(qemu)
```

perhatikan pada R1 berisi value `5` dalam HEX dan pada R2 berisi value `3` dalam HEX dan tentunya pada R3 berisi value `8` dalam HEX yang mana merupakan hasil dari operasi
matematika tambah yang kita instruksikan ke CPU.

Sekarang kita coba debug step by step dengan `lldb` atau `gdb` ke `qemu`,

```bash
(lldb)
Process 1 stopped
* thread #1, stop reason = instruction step into
    frame #0: 0x60010000
->  0x60010000: mov    r1, #5
    0x60010004: mov    r2, #3
    0x60010008: add    r3, r1, r2
    0x6001000c: andeq  r0, r0, r0
Target 0: (No executable module.) stopped.
(lldb)
```

kita dapat melohat kode program kita mulai dibaca dari alamat memori (virtual) 0x60010000, kalau kita coba dump ram/memori kita akan memperlihatkan kode program kita tadi
sudah di load ke dalamnya, dalam `lldb` akan ditampilkan dalam representasi HEX ya.

```bash

(lldb) memory read 0x60010000 --count 50
0x60010000: 05 10 a0 e3 03 20 a0 e3 02 30 81 e0 00 00 00 00  ..... ...0......
0x60010010: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0x60010020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0x60010030: 00 00                                            ..
(lldb)

```

## Menulis program ke binary

kita dapat membuat `.bin` program binary kita dengan bantuan python untuk menuliskan byte ke dalam file.

```python
with open("rom.bin", "wb") as rom:

    binary_data = bytearray([
        0x5, 0x10, 0xA0, 0xE3,
        0x3, 0x20, 0xA0, 0xE3,
        0x2, 0x30, 0x81, 0xE0
    ])
    rom.write(binary_data)
```

namun sebenarnya kita bisa memakai toolset yang sudah disediakan untuk melkaukan kompilasi dari assembly ke machine code termasuk untuk linking dan copy object yang akan
dibahas pada artikel lain di blog ini.

## Penutup
Seleseilah artikel ini, harapnya adalah pembaca sudah mendapatkan intuisi bagaiaman sebenarnya kode mesin di eksekusi di CPU sehingga kita dapat memprogram nya dan sebenarnya tidak terlalu jauh
dengan apa yang kita lakukan sehari hari sebagai penulis kode, justru ini lebih primitif, dari yang saya dapatkan dari mempelajari ini adalah untuk terjun kita harus bolak balik
dari atas ke bawah dari high level ke low level untuk melihat sudut pandang dari kedua sisi sehingga kita paham kenapa begini dan kenapa begitu. Beberapa artikel di blog ini akan membahas
pemograman bahasa mesi juga (assembly) meskipun hanya dasar dan beberap hal yang menarik jika dilihat dari sudut pandang high level.

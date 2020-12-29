# Jarkom_Modul5_Lapres_D14

### KELOMPOK        : D14
ANGGOTA         :

* Muhammad Haris W      (05111840000029)
* Vieri Fath Ayuba      (05111840000153)

**Kelompok kami pembagian subnet menggunakan metode VLSM**

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/topo.png" >

Jumlah IP pada tiap-tiap subnet dan range-nya yang ada pada topologi

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/pembagianip.png" >

Kemudian untuk pembagian pohonnya adalah(puncaknya menggunakan prefix /22 karena menggunakan prefix /23 tidak dapat digunakan untuk membagi 2 subnet /24

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/pohon.png" >

**Selanjutnya dilanjutkan untuk menyetting topologi**

Pertama membuat file ```topo.sh``` di dalam putty dengan konfigurasi

```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &
uml_switch -unix switch4 > /dev/null < /dev/null &
uml_switch -unix switch5 > /dev/null < /dev/null &
uml_switch -unix switch6 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.61 eth1=daemon,,,switch4 eth2=daemon,,,switch5 mem=64M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch4 eth1=daemon,,,switch2 eth2=daemon,,,switch3 mem=64M &
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch5 eth1=daemon,,,switch1 eth2=daemon,,,switch6 mem=64M &

# Server dan Webserver
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &

# Klien
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch6 mem=64M &
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch3 mem=64M &
```

Pada semua UML router, ketikkan nano /etc/sysctl.conf serta uncomment net.ipv4.ip_forward=1 . Untuk mengaktifkan perubahan baru ketikkan ```sysctl -p```.

Lalu setting ```/etc/network/interfaces``` pada masing-masing uml dengan konfigurasi

**SURABAYA (Sebagai Router)**
```
auto eth0
iface eth0 inet static
address 10.151.78.62
netmask 255.255.255.252
gateway 10.151.78.61

auto eth1
iface eth1 inet static
address 192.168.0.1
netmask 255.255.255.252

auto eth2
iface eth2 inet static
address 192.168.0.5
netmask 255.255.255.252
```

**BATU (Sebagai Router)**
```
auto eth0
iface eth0 inet static
address 192.168.0.2
netmask 255.255.255.252
gateway 192.168.0.1

auto eth1
iface eth1 inet static
address 10.151.79.121
netmask 255.255.255.248

auto eth2
iface eth2 inet static
address 192.168.1.1
netmask 255.255.255.0
```

**KEDIRI (Sebagai Router)**
```
auto eth0
iface eth0 inet static
address 192.168.0.6
netmask 255.255.255.252
gateway 192.168.0.5

auto eth1
iface eth1 inet static
address 192.168.0.9
netmask 255.255.255.248

auto eth2
iface eth2 inet static
address 192.168.2.1
netmask 255.255.255.0
```

**MADIUN (Sebagai Web Server)**
```
auto eth0
iface eth0 inet static
address 192.168.0.10
netmask 255.255.255.248
gateway 192.168.0.9
```

**PROBOLINGGO (Sebagai Web Server)**
```
auto eth0
iface eth0 inet static
address 192.168.0.11
netmask 255.255.255.248
gateway 192.168.0.9
```

**MALANG (Sebagai DNS Server)**
```
auto eth0
iface eth0 inet static
address 10.151.79.122
netmask 255.255.255.248
gateway 10.151.79.121
```

**MOJOKERTO (Sebagai DNS Server)**
```
auto eth0
iface eth0 inet static
address 10.151.79.123
netmask 255.255.255.248
gateway 10.151.79.121
```

Kemudian menambahkan file ```route.sh``` di Surabaya saja, karena pada uml untuk 0.0.0.0 sudah otomatis terbuat
```
route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.0.2
route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.0.6
route add -net 192.168.0.8 netmask 255.255.255.248 gw 192.168.0.6
route add -net 10.151.79.120 netmask 255.255.255.248 gw 192.168.0.2
```

Kemudian untuk kofigurasi DHCP Server pada MOJOKERTO dan DHCP RELAY pada KEDIRI, SURABAYA, dan BATU adalah 

Pertama sebelum install harus melakukan ```apt-get update``` terlebih dahulu
 
**Pada DHCP Server**
Jalankan ```apt-get install isc-dhcp-server``` pada uml MOJOKERTO

Lalu setting pada ```/etc/dhcp/dhcpd.conf```, dengan konfigurasi
```
subnet 10.151.79.0 netmask 255.255.255.0 {
}

#SIDOARJO
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.2 192.168.1.254;
    option routers 192.168.1.1;
    option broadcast-address 192.168.1.255;
    option domain-name-servers 202.46.129.2;
    default-lease-time 600;
    max-lease-time 7200;
}

#GRESIK
subnet 192.168.2.0 netmask 255.255.255.0 {
    range 192.168.2.2 192.168.2.254;
    option routers 192.168.2.1;
    option broadcast-address 192.168.2.255;
    option domain-name-servers 202.46.129.2;
    default-lease-time 600;
    max-lease-time 7200;
}
```


Kemudian juga buka ```/etc/default/isc-dhcp-server```, lalu konfigurasi seperti berikut:

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/default-server.png" >

**Pada DHCP Relay**

Jalankan ```apt-get install isc-dhcp-relay```pada uml KEDIRI, SURABAYA, dan BATU

Kemudian buka ```/etc/default/isc-dhcp-relay``` masing-masing dengan:

**BATU**

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/default-batu.png" >

**SURABAYA**

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/default-sby.png" >

**KEDIRI**

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/default-kediri.png" >

Kemudian jalankan ```service isc-dhcp-relay restart``` pada uml KEDIRI, SURABAYA, dan BATU

**Pada Client**

Kemudian buka ```/etc/network/interfaces``` pada GRESIK dan SIDOARJO


**SIDOARJO**
```
auto eth0
iface eth0 inet dhcp
```

**GRESIK**
```
auto eth0
iface eth0 inet dhcp
```

Lalu lakukan ```service networking restart``` pada keduanya :

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/snr-sido.png" >

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/snr-kdr.png" >

Jika mendapatkan IP seperti gambar di atas maka sudah benar untuk seting DHCP Server dan DHCP Relay.


## Jawaban Soal Praktikum Jarkom Modul 5

### Soal No. 1
#### 1. Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan MASQUERADE.

- Pada UML Surabaya membuat file dengan perintah nano soal1.sh yang berisi :

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/1.JPG" >

#### TESTING 

- Jalankan file tersebut dengan perintah bash soal1.sh.
- Untuk mengeceknya, kami tes dengan ping google.com pada UML Surabaya.

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test1.JPG" >


### Soal No. 2
#### 2. Kalian diminta untuk mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.


- Pada UML Surabaya membuat file dengan perintah nano soal2.sh yang berisi :

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/2.JPG" >

- Jalankan file tersebut dengan perintah bash soal2.sh.

#### TESTING :

- Untuk mengeceknya, di UML Malang ketik perintah nc -l -p 22.

- Buka terminal Putty, ketikkan printah nc 10.151.79.122 22. 

- Pada terminal Putty ini ketikan ‘terserah’, tidak akan muncul hasil yang sama di UML Malang karena paket didrop.

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test2A.JPG" >
<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test2B.JPG" >

### Soal No. 3
#### 3. Karena tim kalian maksimal terdiri dari 3 orang, Bibah meminta kalian untuk membatasi DHCP dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

- Pada UML Malang dan Mojokerto membuat file sebagai berikut :

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/3A.JPG" >
<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/3B.JPG" >

- Kemudian, jalankan file tersebut dengan perintah bash no3.sh 

#### TESTING

- Untuk mengeceknya, lakukan ping ke IP Malang atau IP Mojokerto dari 4 UML berbeda.

- Hasilnya nanti salah satu UML tidak bisa melakukan ping ke IP Malang atau IP Mojokerto.

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test3.JPG" >

### Soal No. 4
#### 4. Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat.

- Pada UML Malang membuat file dengan perintah nano no4.sh sebagai berikut :

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/4A.JPG" >

- Kemudian, jalankan file tersebut dengan perintah bash no4.sh.

#### TESTING
- Untuk mengeceknya, lakukan ping ke IP Malang dari UML Sidoarjo.

- Kita bisa mengakses pada hari Senin-Jumat pada pukul 07:00-17:00.

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test4A.JPG" >

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test4B.JPG" >


### Soal No. 5
#### 5. Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya.

- Pada UML Malang membuat file dengan perintah nano no5.sh sebagai berikut :

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/5A.JPG" >

- Kemudian, jalankan file tersebut dengan perintah bash no5.sh.

#### TESTING

- Untuk mengeceknya, lakukan ping ke IP Malang dari UML Sidoarjo.

- Kita bisa mengakses pada pukul 17.00 hingga pukul 07.00 setiap harinya.

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test5A.JPG" >
<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test5B.JPG" >


### Soal No. 6
#### 6. SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80.

- Pada UML Surabaya, Kediri, Batu membuat file dengan perintah nano no6.sh yang berisi:


<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/6.JPG" >
<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/6B.JPG" >
<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/6C.JPG" >

#### TESTING

- Untuk mengeceknya, di UML Probolinggo, ketik perintah nc -l -p 80.
- Buka 2 terminal Putty, ketikkan printah nc 10.151.79.122 22 pada kedua terminal. 

- Di Putty ini ketikan ‘terserah’, nanti akan muncul hasil yang sama di Probolinggo.

- <img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test6.JPG" >




### Soal No. 7
#### 7. Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop.


- Pada UML Surabaya, Malang, Mojokerto membuat file dengan perintah nano no7.sh sebagai berikut :

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/7.JPG" >
<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/7B.JPG" >
<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/7C.JPG" >

- Jalankan file tersebut dengan perintah bash no7.sh.

#### TESTING

- Semua catatan yang didrop akan otomatis muncul pada UML yang bersangkutan.

<img src="https://github.com/hrswcksono/Jarkom_Modul5_Lapres_D14/blob/main/img/test7.JPG" >




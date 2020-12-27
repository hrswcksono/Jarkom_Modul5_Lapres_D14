# Jarkom_Modul5_Lapres_D14

### KELOMPOK        : D14
ANGGOTA         :

* Muhammad Haris W.     (05111840000029)
* Vieri Fath Ayuba      (05111840000153)

**Kelompok kami pembagian subnet menggunakan metode vlsm**

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
Lalu setting ```/etc/network/interfaces``` 

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

**SIDOARJO (Sebagai Client)**
```
auto eth0
iface eth0 inet dhcp
```

**GRESIK (Sebagai Client)**
```
auto eth0
iface eth0 inet dhcp
```

Kemudian menambahkan file ```route.sh``` di Surabaya saja, karena pada uml untuk 0.0.0.0 sudah otomatis terbuat
```
route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.0.2
route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.0.6
route add -net 192.168.0.8 netmask 255.255.255.248 gw 192.168.0.6
route add -net 10.151.79.120 netmask 255.255.255.248 gw 192.168.0.2
```


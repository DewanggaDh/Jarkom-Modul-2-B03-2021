# Jarkom-Modul-2-B03-2021

Nama Anggota :
1. Nouvelli Cornelia (05111940000011)
2. Dewangga Dharmawan (05111940000029)
3. Ahmad Syafiq Aqil Wafi (05111940000089)

## Soal 1 - 7

1. Membuat peta topologi yang menghubungkan berbagai server ke Foosha melaui switch tertentu

DNS Master : EniesLobby

DNS Slave : Water7

Web Server : Skypie

Client : Loguetown dan Alabasta

Semua terhubung ke Foosha

![image](https://user-images.githubusercontent.com/73766205/138793004-d861d30f-7c81-4be9-9225-b33fd2743879.png)

Pertama, Foosha dihubungkan dengan awan NAT. Lalu di dalam konfigurasi IP Foosha dimasukkan kode berikut :
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.178.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.178.2.1
	netmask 255.255.255.0
(Atau nanti dipotong gambarnya habis ini)
```

Lalu, ada dua switch yang dihubungkan ke Foosha. Masing-masing switch dipasang beberapa server. Switch di kiri dipasang server Loguetown dan Alabasta, yang merupakan server client, sedangkan switch di kanan dipasang server EniesLobby (DNS master), Water7 (DNS slave), dan Skypie (Web server).

Masing-masing konfigurasi IP server diisikan dengan kodingan berikut :
```
auto eth0
iface eth0 inet static
	address 192.178.[Nomor switch].[Urutan server dari kiri ke kanan untuk masing-masing switch + 1]
	netmask 255.255.255.0
	gateway 192.178.[Nomor switch].1
```

Lalu, semua node dinyalakan sampai hubungannya berwarna merah.

Di dalam Foosha dibuka web console untuk menyalakan telnet dengan memasukkan :
```
telnet 192.168.0.3 5000
```
Lalu mematikannya dengen CTRL+] jika berhasil.

Berikutnya, di dalam web console Foosha dimasukkan beberapa command :
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.178.0.0/16
cat /etc/resolv.conf
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
Command-command tersebut akan dimasukkan dalam script.sh di dalam Foosha tersebut, yang nanti-nya akan di bash setiap akan dimulai.

Selanjutnya, untuk masing-masing server, dari Loguetown ke Skypie, dibuatkan script.sh berisikan :

```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install bind9 -y
apt-get update
apt-get install dnsutils-y
apt-get update
```

2. Membuat situs domain franky.b03.com dengan alias www.franky.b03.com di dalam folder kaizoku di EniesLobby (DNS utama)

Di dalam EniesLobby, dibukakan /etc/bind/named.conf.local lalu diisikan program berikut :

```
zone "franky.b03.com" {
	type master;
	file "/etc/bind/kaizoku/franky.b03.com";
};
```

Lalu dibuatkan folder kaizoku dan file franky.b03.com di dalamnya. Setelah itu dibuka file yang baru saja dibuat.
```
mkdir /etc/bind/kaizoku
cp /etc/bind/db.local /etc/bind/kaizoku/franky.b03.com
nano /etc/bind/kaizoku/franky.b03.com
```

Di dalam file franky.b03.com, localhost digantikan menjadi franky.b03.com, IP di A digantikan menjadi IP EniesLobby (192.178.2.2), dan ditambahkan satu baris untuk menambahkan CNAME www.franky.b03.com untuk franky.b03.com

(Foto paste here)

Kemudian, di web console EniesLobby, dimasukkan command named -g untuk mengecek kesalahan. Jika semua kesalahan sudah diselesaikan, bisa dimasukkan command service bind9 restart.

(Foto paste here)

Karena menjadi client, nameserver di /etc/resolv.conf di Loguetown dan Alabasta diganti menjadi IP EniesLobby.

(Foto here again)

Lalu, di web console masing-masing, dilakukan ping franky.b03.com dan www.franky.b03.com

(Foto again)

3. Membuat subdomain super.franky.b03.com dengan alias www.super.franky.b03.com ke Skypie

Di buka lagi file franky.b03.com di EniesLobby. Di bawah sendiri ditambahkan kode :
```
super	IN	A	192.178.2.4; (IP Skypie)
```

Setelah itu, dilakukan named -g dan restart bind di EniesLobby.

Lalu, di Loguetown dicoba memasukkan command ping super.franky.b03.com dan host -t A www.super.franky.b03.com

(Foto)

4. Membuat reverse domain utama

Di dalam named.conf.local di dalam EniesLobby, ditambahkan fungsi zona "2.178.192.in-addr-arpa", yang mana merupakan kebalikan dari IP EniesLobby :
```
zone "2.178.192.in-addr.arpa" {
    type master;
    file "/etc/bind/kaizoku/2.178.192.in-addr.arpa";
};
```

Dimasukkan command berikut untuk membuat dan membuka file 2.178.192.in-addr.arpa
```
cp /etc/bind/db.local /etc/kaizoku/2.178.192.in-addr.arpa
nano /etc/bind/kaizoku/2.178.192.in-addr.arpa
```

Selain mengganti localhost menjadi franky.b03.com, dua baris dibawahnya dimasukkan kode :
```
2.178.192.in-addr.arpa	IN	NS	franky.b03.com
2	IN	PTR	franky.b03.com
```

Satu named -g dan restart bind kemudian. Di Loguetown, masukkan command host -t PTR 192.178.2.2

(Foto)

### EniesLobby

![messageImage_1635433392534](https://user-images.githubusercontent.com/73766205/139292599-448ee484-a9ee-4e36-a692-12806f5e11e7.jpg)

![messageImage_1635433426986](https://user-images.githubusercontent.com/73766205/139292740-1a963b29-b46c-4b25-b1fe-0d66b8560843.jpg)

![messageImage_1635433446030](https://user-images.githubusercontent.com/73766205/139292780-5677c5c1-0d23-40fa-adfb-fc13f74a56dd.jpg)

![messageImage_1635433408683](https://user-images.githubusercontent.com/73766205/139292850-9ab64882-45e5-4da5-8da0-6a31a46d9248.jpg)

5. Membuat Water7 sebagai DNS Slave bagi EniesLobby
6. Subdomain www.mecha.franky.b03.com didelegasikan ke Water7 dengan IP ke Skypie
7. Subdomain www.general.mecha.franky.b03.com didelegasikan ke Water7 dengan IP ke Skypie

Water7(Slave)

![messageImage_1635433520421](https://user-images.githubusercontent.com/73766205/139294426-41088a3f-363e-4674-9b09-8788634c2d09.jpg)

![messageImage_1635433477879](https://user-images.githubusercontent.com/73766205/139294431-714844cb-331b-4618-8505-45575be57c21.jpg)

![messageImage_1635433504339](https://user-images.githubusercontent.com/73766205/139294417-1040a747-7fe8-4ecf-a909-1b570e83d9ba.jpg)


## Soal 8

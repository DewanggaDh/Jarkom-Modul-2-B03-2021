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
```
![image](https://user-images.githubusercontent.com/73766205/139516517-7dd03513-de2f-49d9-a121-841c9636735e.png)


Lalu, ada dua switch yang dihubungkan ke Foosha. Masing-masing switch dipasang beberapa server. Switch di kiri dipasang server Loguetown dan Alabasta, yang merupakan server client, sedangkan switch di kanan dipasang server EniesLobby (DNS master), Water7 (DNS slave), dan Skypie (Web server).

Masing-masing konfigurasi IP server diisikan dengan kodingan berikut :
```
auto eth0
iface eth0 inet static
	address 192.178.[Nomor switch].[Urutan server dari kiri ke kanan untuk masing-masing switch + 1]
	netmask 255.255.255.0
	gateway 192.178.[Nomor switch].1
```
Loguetown :

![image](https://user-images.githubusercontent.com/73766205/139516575-05f63ff0-7027-43ca-a774-76d09b296f20.png)

Alabasta :

![image](https://user-images.githubusercontent.com/73766205/139516592-92ca9001-9d0f-40d4-ab71-2ca8360b5475.png)

EniesLobby :

![image](https://user-images.githubusercontent.com/73766205/139516606-d56f4e70-8db8-45f4-9289-6df0f4ca3113.png)

Water7 :

![image](https://user-images.githubusercontent.com/73766205/139516617-6d691806-7f22-4f00-9d5f-005741c05442.png)

Skypie :

![image](https://user-images.githubusercontent.com/73766205/139516631-355cafde-e582-4bb4-bba0-748f78a0cffc.png)

Lalu, semua node dinyalakan sampai hubungannya berwarna merah.

Di dalam Foosha dibuka web console untuk menyalakan telnet dengan memasukkan tulisa di kanan layout GNS3:

![image](https://user-images.githubusercontent.com/73766205/139516675-82047603-271e-46e0-b068-90bedc71691f.png)
```
telnet 192.168.0.3 5000
```
![image](https://user-images.githubusercontent.com/73766205/139516702-7a210e4d-1f06-441f-b244-1d97e9ae3d49.png)

![image](https://user-images.githubusercontent.com/73766205/139516719-8149a0bf-36b3-4b1b-b8b9-22360e6f95be.png)

Lalu mematikannya dengen CTRL+] jika berhasil.

Berikutnya, di dalam web console Foosha dimasukkan beberapa command :
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.178.0.0/16
cat /etc/resolv.conf
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

![image](https://user-images.githubusercontent.com/73766205/139517738-566530a4-8ea1-4226-88b7-c4d1fecc09ef.png)

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

![image](https://user-images.githubusercontent.com/73766205/139517790-847625ff-0d4f-4ca5-a424-2a9f6eef7c35.png)

2. Membuat situs domain franky.b03.com dengan alias www.franky.b03.com di dalam folder kaizoku di EniesLobby (DNS utama)

Di dalam EniesLobby, dibukakan /etc/bind/named.conf.local lalu diisikan program berikut :

```
zone "franky.b03.com" {
	type master;
	notify yes;
	also-notify { 192.178.2.3; };
	allow-transfer { 192.178.2.3; };
	file "/etc/bind/kaizoku/franky.b03.com";
};
```

![image](https://user-images.githubusercontent.com/73766205/139518241-3dbd611b-757b-4af6-856b-8fa6c289d48a.png)


Lalu dibuatkan folder kaizoku dan file franky.b03.com di dalamnya. Setelah itu dibuka file yang baru saja dibuat.
```
mkdir /etc/bind/kaizoku
cp /etc/bind/db.local /etc/bind/kaizoku/franky.b03.com
nano /etc/bind/kaizoku/franky.b03.com
```

Di dalam file franky.b03.com, localhost digantikan menjadi franky.b03.com, IP di A digantikan menjadi IP EniesLobby (192.178.2.2), dan ditambahkan satu baris untuk menambahkan CNAME www.franky.b03.com untuk franky.b03.com

![image](https://user-images.githubusercontent.com/73766205/139518288-68725945-e993-4d06-8e2d-a2792d8583aa.png)

Kemudian, di web console EniesLobby, dimasukkan command named -g untuk mengecek kesalahan. Jika semua kesalahan sudah diselesaikan, bisa dimasukkan command service bind9 restart.

![image](https://user-images.githubusercontent.com/73766205/139518298-e40636ac-d881-4bef-9f56-bc59e56add4f.png)

Karena menjadi client, nameserver di /etc/resolv.conf di Loguetown dan Alabasta diganti menjadi IP EniesLobby.

![image](https://user-images.githubusercontent.com/73766205/139518333-8592b707-636f-4218-b8a3-c4abe0bc586c.png)

Lalu, di web console masing-masing, dilakukan ping franky.b03.com dan www.franky.b03.com

(foto

3. Membuat subdomain super.franky.b03.com dengan alias www.super.franky.b03.com ke Skypie

Di buka lagi file franky.b03.com di EniesLobby. Di bawah sendiri ditambahkan kode :
```
super	IN	A	192.178.2.4; (IP Skypie)
```

Setelah itu, dilakukan named -g dan restart bind di EniesLobby.

Lalu, di Loguetown dicoba memasukkan command ping super.franky.b03.com dan www.super.franky.b03.com

![image](https://user-images.githubusercontent.com/73766205/139518511-d2f0e349-587a-4d50-aa1f-25c5d45fb267.png)

![image](https://user-images.githubusercontent.com/73766205/139518523-0d1271f9-af78-4d8f-a500-d995e635e86e.png)

4. Membuat reverse domain utama

Di dalam named.conf.local di dalam EniesLobby, ditambahkan fungsi zona "2.178.192.in-addr-arpa", yang mana merupakan kebalikan dari IP EniesLobby :
```
zone "2.178.192.in-addr.arpa" {
    type master;
    file "/etc/bind/kaizoku/2.178.192.in-addr.arpa";
};
```
![image](https://user-images.githubusercontent.com/73766205/139518535-9c783774-bcb6-4c3b-9a28-f580974fe2e9.png)

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
![image](https://user-images.githubusercontent.com/73766205/139518551-135d7568-722e-466e-b0ed-2d64c1ca23b3.png)

Satu named -g dan restart bind kemudian. Di Loguetown, masukkan command host -t PTR 192.178.2.2

![image](https://user-images.githubusercontent.com/73766205/139518573-5a99b850-bce6-4aaf-9816-5dba41f6e231.png)

5. Membuat Water7 sebagai DNS Slave bagi EniesLobby

Di dalam Water7, dibuka named.conf.local lalu diisikan dengan kode berikut :

![image](https://user-images.githubusercontent.com/73766205/139519292-e234cef5-4507-41e0-8668-59160060f2ad.png)

Lalu bind9-nya direstart.

Di EniesLobby, bind9 di stop

![image](https://user-images.githubusercontent.com/73766205/139519319-32579de9-e89e-436b-aa48-9b4321a80604.png)

Di Loguetown, ditambahkan IP Water7 di resolv.conf

![image](https://user-images.githubusercontent.com/73766205/139519372-27a83bb6-85b3-4b3b-ad22-fe32e1d0817f.png)

Setelah itu, di ping franky.b03.com di Loguetown

![image](https://user-images.githubusercontent.com/73766205/139519397-e94144b1-9aae-4cc4-8550-ee9faa6e1c77.png)

6. Buat Subdomain www.mecha.franky.b03.com yang didelegasikan ke Water7 dengan IP ke Skypie



7. Buat Subdomain www.general.mecha.franky.b03.com melalui Water7 yang mengarah ke Skypie

Water7(Slave)

![messageImage_1635433520421](https://user-images.githubusercontent.com/73766205/139294426-41088a3f-363e-4674-9b09-8788634c2d09.jpg)

![messageImage_1635433477879](https://user-images.githubusercontent.com/73766205/139294431-714844cb-331b-4618-8505-45575be57c21.jpg)

![messageImage_1635433504339](https://user-images.githubusercontent.com/73766205/139294417-1040a747-7fe8-4ecf-a909-1b570e83d9ba.jpg)


## Soal 8

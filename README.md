# Jarkom-Modul-2-B03-2021

Nama Anggota :
1. Nouvelli Cornelia (05111940000011)
2. Dewangga Dharmawan (05111940000029)
3. Ahmad Syafiq Aqil Wafi (05111940000089)

## Soal-soal

### 1. Membuat peta topologi yang menghubungkan berbagai server ke Foosha melaui switch tertentu

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

### 2. Membuat situs domain franky.b03.com dengan alias www.franky.b03.com di dalam folder kaizoku di EniesLobby (DNS utama)

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

### 3. Membuat subdomain super.franky.b03.com dengan alias www.super.franky.b03.com ke Skypie

Di buka lagi file franky.b03.com di EniesLobby. Di bawah sendiri ditambahkan kode :
```
super	IN	A	192.178.2.4; (IP Skypie)
```

Setelah itu, dilakukan named -g dan restart bind di EniesLobby.

Lalu, di Loguetown dicoba memasukkan command ping super.franky.b03.com dan www.super.franky.b03.com

![image](https://user-images.githubusercontent.com/73766205/139518511-d2f0e349-587a-4d50-aa1f-25c5d45fb267.png)

![image](https://user-images.githubusercontent.com/73766205/139518523-0d1271f9-af78-4d8f-a500-d995e635e86e.png)

### 4. Membuat reverse domain utama

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

### 5. Membuat Water7 sebagai DNS Slave bagi EniesLobby

Di dalam Water7, dibuka named.conf.local lalu diisikan dengan kode berikut :

![image](https://user-images.githubusercontent.com/73766205/139519292-e234cef5-4507-41e0-8668-59160060f2ad.png)

Lalu bind9-nya direstart.

Di EniesLobby, bind9 di stop

![image](https://user-images.githubusercontent.com/73766205/139519319-32579de9-e89e-436b-aa48-9b4321a80604.png)

Di Loguetown, ditambahkan IP Water7 di resolv.conf

![image](https://user-images.githubusercontent.com/73766205/139519372-27a83bb6-85b3-4b3b-ad22-fe32e1d0817f.png)

Setelah itu, di ping franky.b03.com di Loguetown

![image](https://user-images.githubusercontent.com/73766205/139519397-e94144b1-9aae-4cc4-8550-ee9faa6e1c77.png)

### 6. Buat Subdomain www.mecha.franky.b03.com yang didelegasikan ke Water7 dengan IP ke Skypie

Pertama dibuka file franky.b03.com di EniesLobby dan dibawahnya diisi dengan kode :
```
ns1	IN	A	192.178.2.3; (Water7)
mecha	IN	NS	ns1
www.mecha	IN	CNAME	mecha.franky.b03.com
```

![image](https://user-images.githubusercontent.com/73766205/139520952-8db179c1-deee-4a83-a27f-626600c0aa23.png)

Lalu buka file named.conf.options, baik di EniesLobby dan Water7, dan dibuah pada bagian ini sebelum bind9 di-restart :

![image](https://user-images.githubusercontent.com/73766205/139521002-1b97868e-36f3-4489-b3f0-57c8ad8ea284.png)

Di server Water7, diedit named.conf.local sehingga menjadi master dan berisikan file franky.b03.com

![image](https://user-images.githubusercontent.com/73766205/139521123-e8ef364a-eb3a-473c-9ed6-3f881f4e234f.png)

Lalu dibuatkan file mecha.franky.b03.com di folder sunnygo di Water7 dan diedit dengan mecha.franky.b03.com dan ditambahkan IP Skypie dan alias www.mecha.franky.b03.com

![image](https://user-images.githubusercontent.com/73766205/139521285-eaed00f0-d61f-4060-93b1-fd3ff0029d3c.png)

Lalu bind9 di Water7 di-restart lalu dicoba ping di Loguetown

![image](https://user-images.githubusercontent.com/73766205/139521359-ad0a3406-b8be-493f-83dd-f1ea0ba9c660.png)

### 7. Buat Subdomain www.general.mecha.franky.b03.com melalui Water7 yang mengarah ke Skypie

Di Water7, di file mecha.franky.b03.com, ditambahkan kode sebagai berikut :

![image](https://user-images.githubusercontent.com/73766205/139521453-4202adc1-5ede-4b22-98c9-3b3623251252.png)

Setelah itu restart bind9 nya menggunakan command berikut:

```
service bind9 restart
```

Setelah di restart, coba ping melalui Loguetown :

![image](https://user-images.githubusercontent.com/73766205/139521468-4f724c90-9a60-4be8-8510-57762c0963ab.png)

### 8. Membuat webserver dengan DocumentRoot pada /var/www/franky.b03.com

Lakukan perubahan pada pengaturan di franky.b03.com pada EnniesLobby seperti berikut. 

![image](https://user-images.githubusercontent.com/16128257/139560737-0e8596fa-8eed-4d3d-a7b8-e93861f1d2e7.png)

192.178.2.4 merupakan IP Skypie. Hal ini dilakukan untuk memfokuskan server pada Skypie.

Setelah file diubah, restart bind9 menggunakan command berikut.

```
service bind9 restart
```

Setelah itu dilakukan penginstallan apache2 dan php pada Skypie.

```
apt-get update
apt-get install php -y
apt-get install apache2 -y
apt-get install libapache2-mod-php7.0 -y
```

Lalu, pada Server Skypie, dibuatkan file conf apache2 untuk franky.b03.com dengan cara berikut.

```
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/franky.b03.com.conf
```

Lalu dengan nano diubah dan ditambahi bagian berikut.

![image](https://user-images.githubusercontent.com/16128257/139560833-39c036f1-7f95-40a9-855b-a27c98199c0e.png)

Pada command diatas kita menggunakan folder `/var/www/franky.b03.com` sebagai directory server. Untuk itu kita buat folder-nya dan atau langsung ambil data dummy dari github yang telah disediakan lalu rename folder-nya menjadi franky.b03.com.

Pertama kita install `wget` dan `unzip` juga untuk download dan extract file dari github yang telah disediakan.

```
apt-get install wget -y
apt-get install unzip -y
```

```
cd /var/www
wget https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom/raw/main/franky.zip
unzip franky.zip
mv franky franky.b03.com
```

![image](https://user-images.githubusercontent.com/16128257/139561120-dd2dca4d-9de7-4acf-981d-a1c2b9b5fe15.png)

Lalu aktifkan konfigurasi apache franky.b03.com.conf menggunakan command berikut.

```
a2ensite franky.b03.com
```

Setelah itu restart apache2 nya.

```
service apache2 restart
```

Lalu pada client Loguetown atau Alabasta install Lynx atau curl. Namun kami akan coba menggunakan curl.

```
apt-get update
apt-get install curl
```

Lalu visit franky.b03.com dan www.franky.b03.com dengan menggunakan curl dan lihat hal yang terjadi.

![image](https://user-images.githubusercontent.com/16128257/139561136-33041024-8650-4a04-a3af-fa3e6d2df742.png)

Dapat dilihat bahwa curl success mengembalikan data pada server Skypie yang telah di-konfigurasikan.

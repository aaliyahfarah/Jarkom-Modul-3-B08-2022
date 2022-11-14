# Jarkom-Modul-3-B08-2022

## Anggota Kelompok
<table>
 	<tr>
 		<td> Nama </td>
 		<td> NRP</td>
 	</tr>
 	<tr>
 		<td> Aaliyah Farah Adibah </td>
 		<td> 5025201070 </td>
 	</tr>
  <tr>
 		<td> Rafael Asi Kristanto Tambunan </td>
 		<td> 5025201168 </td>
 	</tr>
  <tr>
 		<td> Sejati Bakti Raga </td>
 		<td> 5025201007 </td>
 	</tr>
 </table>
 
 ## Daftar Isi
  + [Soal 1](#soal-1)
  + [Soal 2](#soal-2)
  + [Soal 3](#soal-3)
  + [Soal 4](#soal-4)
  + [Soal 5](#soal-5)
  + [Soal 6](#soal-6)
  + [Soal 7](#soal-7)
  + [Soal 8](#soal-8)
  + [Soal 9](#soal-9)
  + [Soal 10](#soal-10)
  + [Soal 11](#soal-11)
  + [Soal 12](#soal-12)
  + [Kendala](#kendala)
  
## Soal 1
 <img alt="topologi-soal" src="pic/topologi-soal.png">
 
 ***Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, 
 Westalis sebagai DHCP Server, Berlint sebagai Proxy Server (1)***<br><br>
 
 Pertama-tama, buat topologinya terlebih dahulu.
 <img alt="topologi" src="pic/topologi.png">
 
 Lalu konfigurasi setiap nodenya<br>
 
 **Ostania**   
 
```
auto eth0
iface eth0 inet dhcp
auto eth1
  iface eth1 inet static
	 address 10.7.1.1
	 netmask 255.255.255.0
 
 auto eth2
 iface eth2 inet static
	  address 10.7.2.1
	  netmask 255.255.255.0
 
 auto eth3
 iface eth3 inet static
   address 10.7.3.1
	  netmask 255.255.255.0
 ```
 
 **SSS**
 
 ```
 auto eth0
 iface eth0 inet dhcp
 ```
 
 **Garden**
 
 ```
 auto eth0
 iface eth0 inet dhcp
 ```
 
 **WISE**
 
 ```
 auto eth0
 iface eth0 inet static
	  address 10.7.2.2
	  netmask 255.255.255.0
	  gateway 10.7.2.1
 ```
 
 **Berlint**
 
 ```
 auto eth0
 iface eth0 inet static
	  address 10.7.2.3
	  netmask 255.255.255.0
	  gateway 10.7.2.1
 ```
 
  **Westalis**
 
 ```
 auto eth0
 iface eth0 inet static
	  address 10.7.2.4
	  netmask 255.255.255.0
	  gateway 10.7.2.1
 ```
 
  **Eden**
 
 ```
auto eth0
iface eth0 inet dhcp
	hwaddress ether a6:09:e5:e8:1a:14
 ```
 
 **NewstonCastle**
 
 ```
 auto eth0
auto eth0
iface eth0 inet dhcp
  ```
  
  **KemonoPark**
 
 ```
 auto eth0
 iface eth0 inet dhcp
  ```
 
 Buat node sesuai dengan kriterianya:<br>
 
```shell
# WISE
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install bind9 -y
# Westalis
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-server -y
# Berlint
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install squid -y
```

Selain, itu untuk menjadikan Westalis sebagai DHCP Server, kita perlu menambahkan konfigurasi pada node Westalis pada file `/etc/default/isc-dhcp-server` yang kita buat konfigurasinya pada file temporary yaitu `no1.sh` dengan ditambahkan `INTERFACES=\"eth0\"" > /etc/default/isc-dhcp-server`

```shell
# Defaults for isc-dhcp-server initscript
# sourced by /etc/init.d/isc-dhcp-server
# installed at /etc/default/isc-dhcp-server by the maintainer scripts
#
# This is a POSIX shell fragment
#
# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPD_CONF=/etc/dhcp/dhcpd.conf
# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPD_PID=/var/run/dhcpd.pid
# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACES=\"eth0\"" > /etc/default/isc-dhcp-server
```

Setelah itu, kita jalankan script `no1.sh` pada node Westalis yang berisi:

```shell
cp /root/isc-dhcp-server-1 /etc/default/isc-dhcp-server
service isc-dhcp-server restart
```

Hal ini membuat masing-masing node memiliki server tersendiri.

   
## Soal 2

***dan Ostania sebagai DHCP Relay (2). Loid dan Franky menyusun peta tersebut 
dengan hati-hati dan teliti***<br><br> 

Ostania -> DHCP Relay 

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.8.0.0/16
cat /etc/resolv.conf
apt-get update
echo "" | apt-get install isc-dhcp-relay -y
```

Setelah itu, lakukan konfigurasi pada node Ostania pada file `/etc/default/isc-dhcp-relay` yang kita buat konfigurasinya pada file temporary yaitu `no2.sh` yang berisi:

```shell
echo -e '
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="10.7.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
' > /etc/default/isc-dhcp-relay

service isc-dhcp-relay start
```

## Soal 3

***Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu:***

***1. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.***

***2. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 
dan [prefix IP].1.120 - [prefix IP].1.155 (3)*** <br><br> 

Pertama konfigurasi DHCP Relay pada Ostania <br>
Lakukan konfigurasi pada Ostania dengan melakukan edit file /etc/default/isc-dhcp-relay dengan konfigurasi berikut:

```
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="10.45.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth3 eth2"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```

Lalu konfigurasi DHCP Server pada Westalis<br>

Membuat Westalis menjadi DHCP Server. Karena Westais Terhubung dengan Ostania melalui eth0 sehingga lakukan konfigurasi pada file /etc/default/isc-dhcp-server sebagai berikut:

```
# Defaults for isc-dhcp-server initscript
# sourced by /etc/init.d/isc-dhcp-server
# installed at /etc/default/isc-dhcp-server by the maintainer scripts

#
# This is a POSIX shell fragment
#

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPD_CONF=/etc/dhcp/dhcpd.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPD_PID=/var/run/dhcpd.pid

# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACES=\"eth0\"" > /etc/default/isc-dhcp-server
```

Setelah itu, lakukan restart DHCP Server di Westalis

```
service isc-dhcp-server restart
```

Lakukan konfigurasi di DHCP Server untuk rentang IP yang akan diberikan pada file /etc/dhcp/dhcpd.conf dengan cara:

```
subnet 10.7.2.0 netmask 255.255.255.0 {
}
subnet 10.7.1.0 netmask 255.255.255.0 {
    range  10.7.1.50 10.7.1.88;
    range  10.7.1.120 10.7.1.155;
    option routers 10.7.1.1;
    option broadcast-address 10.7.1.255;
    option domain-name-servers 10.7.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}
```

 **TESTING**
 <img alt="testing3" src="pic/testing3.png">

## Soal 4

***3. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 
dan [prefix IP].3.60 - [prefix IP].3.85 (4)***<br><br> 

Lakukan konfigurasi di DHCP Server untuk rentang IP yang akan diberikan pada file /etc/dhcp/dhcpd.conf dengan cara menambahkan:

```
subnet 10.7.3.0 netmask 255.255.255.0 {
}
subnet 10.7.3.0 netmask 255.255.255.0 {
    range  10.7.3.10 10.7.3.30;
    range  10.7.3.60 10.7.3.85;
    option routers 10.7.3.1;
    option broadcast-address 10.7.3.255;
    option domain-name-servers 10.7.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}
```

 **TESTING**
 <img alt="testing4" src="pic/testing4.png">

## Soal 5

***4. Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS 
tersebut (5)***<br><br>

Untuk client mendapatkan DNS dari WISE diperlukan konfigurasi pada file /etc/dhcp/dhcpd.conf dengan option domain-name-servers 10.45.2.2

Supaya semua client dapat terhubung internet pada WISE diberikan konfigurasi pada file /etc/bind/named.conf.options dengan

```
options {
        directory \"/var/cache/bind\";

        forwarders {
                8.8.8.8;
                8.8.8.4;
        };

        // dnssec-validation auto;
        allow-query { any; };
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

 **TESTING**
 <img alt="testing5" src="pic/testing5.png">

## Soal 6

***5. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 
5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal 
yang dialokasikan untuk peminjaman alamat IP selama 115 menit. (6)***<br><br>

Pada subnet interface switch 1 dan 3 ditambahkan konfigurasi berikut pada file `/etc/dhcp/dhcpd.conf`

```
subnet 10.7.1.0 netmask 255.255.255.0 {
    ...
    default-lease-time 300; 
    max-lease-time 6900;
    ...
}
subnet 10.7.3.0 netmask 255.255.255.0 {
    ...
    default-lease-time 600;
    max-lease-time 6900;
    ...
}
```

 **TESTING**
 <img alt="testing6" src="pic/testing6.png">

## Soal 7

***Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi 
dengan alamat IP yang tetap dengan IP [prefix IP].3.13 (7)***<br><br>

Pada Eden, menambahkan konfigurasi untuk fixed address pada `/etc/dhcp/dhcpd.conf`

```
host Eden {
    hardware ethernet a6:09:e5:e8:1a:14;
    fixed-address 10.7.3.13;
}
```
Setelah itu tidak lupa untuk mengganti konfigurasi pada file `/etc/network/interfaces` dengan
```
auto eth0
	iface eth0 inet dhcp
	hwaddress ether a6:09:e5:e8:1a:14
```

 **TESTING**
 <img alt="testing7" src="pic/testing7.png">

## Soal 8

***SSS, Garden, dan Eden digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin
keamanannya, juga untuk mencegah kebocoran data. Pada Proxy Server di Berlint, Loid berencana 
untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan 
Berlint sebagai HTTP & HTTPS proxy. Adapun kriteria pengaturannya adalah sebagai berikut:***

***1. Client hanya dapat mengakses internet diluar (selain) hari & jam kerja 
(senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)***<br><br>

**Penambahan ACL Waktu dan Konfigurasi Akses Berdasarkan ACL Waktu**
**
Buat file `no8-9.sh` dan isikan potongan teks berikut:
- squid8-9.conf
  
  ```shell
    echo -e '
	.loid-work.com
	.franky-work.com
	' > /etc/squid/domain.acl


	echo -e '
	acl AVAILABLE_1 time MTWHF 00:00-07:59
	acl AVAILABLE_2 time MTWHF 17:01-23:59
	acl AVAILABLE_3 time AS 00:00-23:59

	acl WORK_TIME time MTWHF 08:00-17:00

	acl RESTRICTED_DOMAIN dstdomain "/etc/squid/domain.acl"
	' > /etc/squid/acl.conf

	echo -e '
	include /etc/squid/acl.conf

	http_port 8080
	visible_hostname Berlint

	http_access deny RESTRICTED_DOMAIN AVAILABLE_1
	http_access deny RESTRICTED_DOMAIN AVAILABLE_2
	http_access deny RESTRICTED_DOMAIN AVAILABLE_3

	http_access allow AVAILABLE_1 all
	http_access allow AVAILABLE_2 all
	http_access allow AVAILABLE_3 all

	http_access allow RESTRICTED_DOMAIN WORK_TIME
	
	http_access deny all

	' > /etc/squid/squid.conf

	service squid restart
  ```
  

Maka, ketika di test pada client dengan `lynx http://its.ac.id` dan `lynx https://its.ac.id` akan menghasilkan seperti ini:
- Senin (10.00) -> tidak dapat diakses
![201596650-3accc237-a0a4-4dbc-bb78-0702e9518810](https://user-images.githubusercontent.com/88140623/201692744-64e7720e-79e2-47eb-8eb4-c9ce2bd0351d.png)
- Senin (20.00) dan Sabtu (10.00) -> dapat diakses
![201596969-0da0f06c-da91-4995-9eb7-86b2d9f2f5f5](https://user-images.githubusercontent.com/88140623/201692769-67c316d8-1a9f-44b7-bf97-be8d1f9d8239.png)

## Soal 9

***2. Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses 
domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)***<br><br>

Pada Node Wise, buat isi file `named.conf.local` kemudian buat directory baru dengan perintah `mkdir /etc/bind/wise` dan buat file baru lagi pada directory wise dengan nama `loid-work.com` dan `franky-work.com` kemudian isikan syntax berikut :

```
    shell
    # /etc/bind/named.conf.local
    zone "loid-work.com" {
        type master;
        file "/etc/bind/wise/loid-work.com";
        allow-transfer { 10.7.3.13; };
    };

    zone "franky-work.com" {
        type master;
        file "/etc/bind/wise/franky-work.com";
        allow-transfer { 10.7.3.13; };
    };

    # /etc/bind/wise/loid-work.com
    ;
    ; BIND data file for local loopback interface
    ;
    $TTL    604800
    @       IN      SOA     loid-work.com. root.loid-work.com. (
                            2022100601      ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      loid-work.com.
    @       IN      A       10.7.3.13
    www     IN      CNAME   loid-work.com.

    # /etc/bind/wise/franky-work.com
    ;
    ; BIND data file for local loopback interface
    ;
    $TTL    604800
    @       IN      SOA     franky-work.com. root.franky-work.com. (
                            2022100601      ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      franky-work.com.
    @       IN      A       10.7.3.13
    www     IN      CNAME   franky-work.com.
```

Setelah selesai konfigurasi tersebut berhasil, lakukan copy file dari semua file tersebut dengan `cp /root/named-8.conf.local /etc/bind/named.conf.local`, `cp /root/loid-work-8.com /etc/bind/wise/loid-work.com`, `cp /root/franky-work-8.com /etc/bind/wise/franky-work.com`. Kemudian restart bind9 dengan `service bind9 restart`. <br> 

Buat isi file `apache2.conf` , `loid-work.com.conf` , `franky-work.com.conf` dengan copy semua file dengan `cp /root/apache2-8.conf /etc/apache2/apache2.conf` , `cp /root/loid-default-8.conf /etc/apache2/sites-available/loid-work.com.conf` , `cp /root/franky-default-8.conf /etc/apache2/sites-available/franky-work.com.conf` .

```
   apt-get install apache2 -y
   apt-get install php -y
   apt-get install libapache2-mod-php7.0 -y

   # /etc/apache2/apache2.conf
   ...
   ServerName 10.8.3.13
   
   # /etc/apache2/sites-available/loid-work.com.conf
   <VirtualHost *:80>
        ServerName loid-work.com
        ServerAlias www.loid-work.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/loid-work.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>

   # /etc/apache2/sites-available/franky-work.com.conf
   <VirtualHost *:80>
        ServerName franky-work.com
        ServerAlias www.franky-work.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky-work.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
```

Setelah konfigurasi tersebut, aktifkan domain dengan a2ensite dengan perintah `a2ensite loid-work.com` dan `a2ensite franky-work.com`. Kemudian buat directory baru dengan `mkdir /var/www/loid-work.com` dan `mkdir /var/www/franky-work.com` . Setelah itu masukan konfigurasi syntax dibawah ini:

```
   # /var/www/loid-work.com/index.php
   <?php
        echo "Ini domain loid-work";
   ?>
   
   # /var/www/franky-work.com/index.php
   <?php
        echo "Ini domain franky-work";
   ?>
```

Pada Berlint, tambahkan konfigurasi pada file `acl-1.conf`

```
  acl CAN_ACCESS_1 time MTWHF 00:00-07:59
  acl CAN_ACCESS_2 time MTWHF 17:01-23:59
  acl CAN_ACCESS_3 time AS 00:00-23:59

  acl WORK_HOUR time MTWHF 08:00-17:00
  acl CERTAIN_DOMAIN dstdomain .loid-work.com .franky-work.com
```

pada file `squid8-9.conf` tambahkan:

```
  include /etc/squid/acl.conf

  http_port 8080
  visible_hostname Berlint

  http_access deny CERTAIN_DOMAIN CAN_ACCESS_1
  http_access deny CERTAIN_DOMAIN CAN_ACCESS_2
  http_access deny CERTAIN_DOMAIN CAN_ACCESS_3
  
  http_access allow CAN_ACCESS_1
  http_access allow CAN_ACCESS_2
  http_access allow CAN_ACCESS_3
  
  http_access allow CERTAIN_DOMAIN WORK_HOUR
```

Syntax `http_access allow CERTAIN_DOMAIN WORK_HOUR` akan memperbolehkan domain hanya diakses di jam kerja. Setelah itu restart apache2 dengan `service apache2 restart`.

 **TESTING**
 <img alt="testing9" src="pic/testing9.png">

## Soal 10

***3. Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. 
(Contoh web HTTP: http://example.com)***<br><br>

 **TESTING**
 <img alt="testing10" src="pic/testing10.png">

## Kendala
+ Aaliyah Farah Adibah

	1. Hanya bisa mengerjakan hingga no. 7 
	2. GNS yang mudah error
	3. Sempat fail di restart DHCP Server hingga perlu run semua node dari awal
	
+ Rafael Asi Kristanto Tambunan

	1. Penggunaan GNS yang masih baru
	2. GNS yang sering error saat digunakan

+ Sejati Bakti Raga

	1. Pemahaman GNS yang masih kurang
	2. GNS dan VM error ditengah pengerjaan



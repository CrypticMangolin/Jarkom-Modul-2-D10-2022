# Jarkom-Modul-2-D10-2022
Repositori untuk Laporan Resmi praktikum mata kuliah Jarkom 2022 Modul 2: DNS dan WebServer

## Anggota Kelompok:
|Nama     |NRP    |
|----------|-------|
|Hasna Lathifah Purbaningdyah|5025201108|
|Anggito Anju Hartawan Manalu|5025201216|
|Hidayatullah|5025201031|

# Nomor 1
Membuat topologi sesuai gambar pada soal shift

<img width="466" alt="image" src="https://user-images.githubusercontent.com/80145586/198836484-d97bdeda-d7b9-4ff8-a4eb-2196d2541d8f.png">

IP Address dari masing-masing node adalah:

    - Ostania : 10.20.1.1	| Router
    - SSS     : 10.20.1.2	| Client
    - Garden  : 10.20.1.3	| Client
    - WISE    : 10.20.2.2	| DNS Master
    - Berlint : 10.20.3.2	| DNS Slave
    - Eden    : 10.20.3.3	| Web Server

## Isi File konfigurasi `no1.sh`

### Pada node Ostania
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.20.0.0/16
cat /etc/resolv.conf
```

### Pada semua node kecuali Ostania
```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
## Hasil ping google

<img width="468" alt="image" src="https://user-images.githubusercontent.com/80145586/198836494-82945eff-7c7a-4f60-a30f-167ae2d65ec8.png">

# Nomor 2
Membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise

##  Pada node Wise 
Install bind9 dengan `apt-get install bind9`

Konfigurasi node agar menjadi DNS MASTER dengan menambahkan zone baru `wise.d10.com` dengan type master pada file `/etc/bind/named.conf.local`
```bash
zone "wise.d10.com" {
        type master;
        file "/etc/bind/wise/wise.d10.com";
};
```
Lalu buat folder baru `/etc/bind/wise` untuk menyimpan konfigurasi dns website `wise.d10.com`
alias `www.wise.d10.com` juga dapat ditambahkan di dalam file konfigurasi `/etc/bind/wise/wise.d10.com`
```bash
$TTL    604800
@       IN      SOA     wise.d10.com. root.wise.d10.com. (
                     2022102501         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      wise.d10.com.
@       IN      A       10.20.3.2
www     IN      CNAME   wise.d10.com.
@       IN      AAAA    ::1
```
Restart bind9 pada DNS Master serta aliasnya yang sudah dibuat

## Pada node Client (SSS dan Garden)

Mengatur agar IP mengarah ke WISE, Berlint, dan Eden

Isi file konfigurasi no2.sh

```bash
cp /root/config2.txt /etc/resolv.conf

# Isi file config2.txt untuk file /etc/resolv.conf :

# nameserver 10.20.3.2
# nameserver 10.20.2.3
# nameserver 10.20.2.2
```

## Hasil ping `wise.d10.com` pada node client
Ping website `wise.d10.com` 

<img width="468" alt="image" src="https://user-images.githubusercontent.com/80145586/198836539-f0feb214-91d2-4d98-bd3b-f521c8aa6351.png">

## Hasil ping `www.wise.d10.com` pada node client

<img width="465" alt="image" src="https://user-images.githubusercontent.com/80145586/198836553-56705325-f61b-452d-96c2-03b9601bd955.png">

# Nomor 3
membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden

## Isi file konfigurasi `no3.sh`

### Pada node WISE
```bash
cp /root/config3.txt /etc/bind/wise/wise.d10.com

# Isi file /root/config2.txt untuk file /etc/bind/wise/wise.d10.com :

# -------------------------------------------------

#;
#; BIND data file for local loopback interface
#;
#$TTL    604800
#@       IN      SOA     wise.d10.com. root.wise.d10.com. (
#                              2022102501                ; Serial
#                         604800         ; Refresh
#                          86400         ; Retry
#                        2419200         ; Expire
#                         604800 )       ; Negative Cache TTL
#;
#@       IN      NS      wise.d10.com.
#@       IN      A       10.20.3.2
#www     IN      CNAME   wise.d10.com.
#eden    IN      A       10.20.2.3
#www.eden IN     A       10.20.2.3
#@       IN      AAAA    ::1

# -------------------------------------------------

service bind9 restart
```
## Hasil `ping wise.d10.com -c 5` pada node client (SSS atau Garden)
<img width="465" alt="image" src="https://user-images.githubusercontent.com/80145586/198836591-4ef00a33-790e-4830-af5f-a858a022a123.png">

## Hasil `ping www.wise.d10.com -c 5` pada node client (SSS atau Garden)
<img width="468" alt="image" src="https://user-images.githubusercontent.com/80145586/198836599-f9833185-dbd4-4bd3-9ff3-b422fa614d14.png">

# Nomor 4
Membuat reverse domain untuk domain utama

## Pada Node WISE

Menambahkan zone baru dengan reverse dari 3 byte awal IP Wise
```bash
zone `3.20.10.in-addr.arpa` {
      type master;
      file `/etc/bind/wise/3.20.10.in-addr.arpa`;
};
```
Lalu menambahkan dns record di dalam file `/etc/bind/wise/3.20.10.in-addr.arpa`
```bash
$TTL    604800
@       IN      SOA     wise.d10.com. root.wise.d10.com. (
                             2022102501                ; Serial
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
3.20.10.in-addr.arpa.   IN      NS      wise.d10.com.
2                       IN      PTR     wise.d10.com.
```
Restart bind9

Mengetes pada node client menggunakan command `host -t PTR [IP Wise]`
<img width="460" alt="image" src="https://user-images.githubusercontent.com/80145586/198836611-753d7030-d842-45dc-a221-a8ebd488094e.png">

#Nomor 5
Membuat Berlint sebagai DNS Slave untuk domain utama
## Pada node WISE
Menambahkan `also-notify { [IP Berlint]; };` dan `allow-transfer { [IP Belint]; };` pada `zone “wise.d10.com”’ dalam file `/etc/bind/named.conf.local`

### Isi file konfigurasi `no5.sh`
```bash
cp /root/config5.txt /etc/bind/named.conf.local

# Isi file /root/config5.txt untuk file /etc/bind/named.conf.local :

# -------------------------------------------------

#//
#// Do any local configuration here
#//

#// Consider adding the 1918 zones here, if they are not used in your
#// organization
#//include `/etc/bind/zones.rfc1918`;

#zone `wise.d10.com` {
#        type master;
#        notify yes;
#        also-notify { 10.20.2.2; };
#        allow-transfer { 10.20.2.2; };
#        file `/etc/bind/wise/wise.d10.com`;
#};

#zone `3.20.10.in-addr.arpa` {
#    type master;
#    file `/etc/bind/wise/3.20.10.in-addr.arpa`;
#};

# -------------------------------------------------

#NB : tanda ` adalah titik dua (")

service bind9 restart
```
## Pada node Berlint
Install bind9, kemudian menambahkan
```bash
zone `wise.d10.com` {
    type slave;
    masters { 10.20.3.2; };
    file `/var/lib/bind/wise.d10.com`;
};
```
Pada file `/etc/bind/named.conf.local`

### Isi file konfigurasi `no5.sh`
```bash
apt-get update
apt-get install bind9 -y

cp /root/config5.txt /etc/bind/named.conf.local

# Isi file /root/config5.txt untuk file /etc/bind/named.conf.local :

# -------------------------------------------------

#//
#// Do any local configuration here
#//

#// Consider adding the 1918 zones here, if they are not used in your
#// organization
#//include `/etc/bind/zones.rfc1918`;

#zone `wise.d10.com` {
#    type slave;
#    masters { 10.20.3.2; };
#    file `/var/lib/bind/wise.d10.com`;
#};

# -------------------------------------------------

#NB : tanda ` adalah petik dua (")

service bind9 restart
```
## Hasil testing
Command `service bind9 stop` pada node WISE, kemudian `ping wise.d10.com -c 5` pada node client
<img width="472" alt="image" src="https://user-images.githubusercontent.com/80145586/198836629-bcac2d1b-eacf-44a7-a171-1b876bfc13fc.png">

<img width="431" alt="image" src="https://user-images.githubusercontent.com/80145586/198836635-53ac3c17-2758-4ac9-b340-1b082eca8cbd.png">

# Nomor 6
Buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com 

## Pada node Wise 
Tambahkan alias baru pada file `etc/bind/wise/wise.d10.com` yang menuju ke IP Eden
```bash
ns1      IN     A       10.20.2.3
operation IN    NS      ns1
```
Comment `dnssec-validation auto` dan tambahkan line berikut pada `/etc/bind/named.conf.options`
```
//dnssec-validation auto
allow-query{any;};
```
Restart bind9

## Pada node Berlint
Comment `dnssec-validation auto` dan tambahkan line berikut pada `/etc/bind/named.conf.options`
```bash
//dnssec-validation auto
allow-query{any;};
```
Buat zone baru untuk subdomain `operation.wise.d10.com`
```bash
zone `operation.wise.d10.com` {
   type master;
   file `/etc/bind/operation/operation.wise.d10.com`;
};
```
Buat folder operation pada `etc/bind/operation` dan buat file konfigurasi untuk subdomainnya
```bash
$TTL    604800
@       IN      SOA     operation.wise.d10.com. root.operation.wise.d10.com. (
                             2022102501                ; Serial
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      operation.wise.d10.com.
@       IN      A       10.20.2.3
www     IN      CNAME   operation.wise.d10.com.
```

## Hasil ping subdomain
<img width="465" alt="image" src="https://user-images.githubusercontent.com/80145586/198836648-8703df6f-f093-4684-9162-d438cbb6685c.png">

#Nomor 7
Membuat subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden

## Pada node Berlint
Menambahkan subdomain `strix` dan `www.strix` pada file `/etc/bind/operation/operation.wise.d10.com`

### Isi file konfigurasi `no7.sh`
```bash
cp /root/config7.txt /etc/bind/operation/operation.wise.d10.com

# Isi file /root/config7.txt untuk file /etc/bind/operation/operation.wise.d10.com :

# -------------------------------------------------

#;
#; BIND data file for local loopback interface
#;
#$TTL    604800
#@       IN      SOA     operation.wise.d10.com. root.operation.wise.d10.com. (
#                              2022102501                ; Serial
#                         604800         ; Refresh
#                          86400         ; Retry
#                        2419200         ; Expire
#                         604800 )       ; Negative Cache TTL
#;
#@       IN      NS      operation.wise.d10.com.
#@       IN      A       10.20.2.3
#www     IN      CNAME   operation.wise.d10.com.
#strix   IN      A       10.20.2.3
#www.strix IN    A       10.20.2.3

# -------------------------------------------------

service bind9 restart
```
## Hasil testing
Hasil `ping strix.operation.wise.d10.com -c 5`<img width="468" alt="image" src="https://user-images.githubusercontent.com/80145586/198836666-d5a1b287-cb64-4720-aa47-a3844795ef0a.png"> pada client
<img width="457" alt="image" src="https://user-images.githubusercontent.com/80145586/198836656-b8734cc9-d729-450a-9966-07499b045db2.png">

Hasil `ping www.strix.operation.wise.d10.com -c 5` pada client
<img width="457" alt="image" src="https://user-images.githubusercontent.com/80145586/198836689-fbecc053-f1d9-4ab7-8236-3fc221679b3b.png">

## Kendala/error
Pada saat demo terjadi error ip address tidak mengarah ke ip Eden, karena lupa me-run file `no2.sh` pada node client sehingga client belum tersambung ke IP WISE, Berlint, dan Eden

#Nomor 8
Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com

## Pada node WISE
###Isi file konfigurasi `no8.sh`
```
apt-get update
apt-get install lynx

apt-get install apache2
service apache2 start

apt-get install php

apt-get install libapache2-mod-php7.0

service apache2 start

cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/wise.d10.com.conf

mkdir /var/www/wise.d10.com

cp /root/config8.txt /etc/apache2/sites-available/wise.d10.com.conf

# Isi file /root/config8.txt untuk file /etc/apache2/sites-available/wise.d10.com.conf :

# -------------------------------------------------

#<VirtualHost *:80>
#        ServerAdmin webmaster@localhost
#        DocumentRoot /var/www/wise.d10.com
#        ServerName wise.d10.com
#        ServerAlias www.wise.d10.com

#        ErrorLog ${APACHE_LOG_DIR}/error.log
#        CustomLog ${APACHE_LOG_DIR}/access.log combined
#</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

# -------------------------------------------------

a2ensite wise.d10.com

service apache2 restart

cp /root/config81.txt /var/www/wise.d10.com/index.php

# Isi file /root/config81.txt untuk file /var/www/wise.d10.com/index.php

# -------------------------------------------------

#<?php
#        phpinfo();
#?>

# -------------------------------------------------

service apache2 restart
```
## Hasil testing
Hasil command `lynx wise.d10.com` pada client
<img width="466" alt="image" src="https://user-images.githubusercontent.com/80145586/198836694-5efc0aa9-234b-4344-aded-a3b7b6afea9b.png">

Hasil command `lynx www.wise.d10.com` pada client
![Uploading image.png…]()

## Kendala/error
Pada saat demo terjadi error karena lupa me-run file `no2.sh` pada node client sehingga client belum tersambung ke IP WISE, Berlint, dan Eden

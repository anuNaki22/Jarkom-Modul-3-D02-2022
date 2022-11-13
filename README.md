# Jarkom-Modul-3-D02-2022
## Anggota Kelompok D02
| NRP | Nama | Kontribusi |
| :---:        |     :---:           | :---: |
| 5025201082   | Farrel Emerson      | proxy server    |
| 5025201087   | Daniel Hermawan     | proxy server     |
| 5025201003   | Rahmat Faris Akbar  |   1-7    | 


### Gambar Topologi
![image](https://user-images.githubusercontent.com/99629909/201476404-18dd4c7e-e436-4ea9-aaae-68aed70ebd47.png)

Lakukan setting network masing-masing node dengan fitur `Edit network configuration` dengan setting sebagai berikut:
- Ostania 
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
 address 10.16.1.1
 netmask 255.255.255.0

auto eth2
iface eth2 inet static
 address 10.16.2.1
 netmask 255.255.255.0

auto eth3
iface eth3 inet static
 address 10.16.3.1
 netmask 255.255.255.0
```

- Wise 
```
auto eth0
iface eth0 inet static
 address 10.16.2.2
 netmask 255.255.255.0
 gateway 10.16.2.1
```

- Berlint 
```
auto eth0
iface eth0 inet static
 address 10.16.2.3
 netmask 255.255.255.0
 gateway 10.16.2.1
```

- Westalis 
```
auto eth0
iface eth0 inet static
 address 10.16.2.4
 netmask 255.255.255.0
 gateway 10.16.2.1
```

- SSS 
```
#auto eth0
#iface eth0 inet static
# address 10.16.1.2
# netmask 255.255.255.0
# gateway 10.16.1.1
 auto eth0
 iface eth0 inet dhcp
```

- Garden
```
auto eth0
iface eth0 inet static
 address 10.16.1.3
 netmask 255.255.255.0
 gateway 10.16.1.1
# auto eth0
# iface eth0 inet dhcp
```

- Eden
```
# auto eth0
# iface eth0 inet static
# address 10.16.3.2
# netmask 255.255.255.0
# gateway 10.16.3.1
 auto eth0
 iface eth0 inet dhcp
 hwaddress ether f6:18:ab:b8:88:1d
```

- NewstonCastle 
```
#auto eth0
# iface eth0 inet static
# address 10.16.3.3
# netmask 255.255.255.0
# gateway 10.16.3.1
auto eth0
iface eth0 inet dhcp
```

- KemonoPark 
```
#auto eth0
#iface eth0 inet static
# address 10.16.3.4
# netmask 255.255.255.0
# gateway 10.16.3.1
auto eth0
iface eth0 inet dhcp
```


### 1
> Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server
### Penyelesaian
Kita dapat melakukan penginstallan masing-masing kebutuhan node yang dilakukan langsung di file `/root/.bashrc` dengan format sebagai berikut:

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

Selain, itu untuk menjadikan Westalis sebagai DHCP Server, kita perlu menambahkan konfigurasi pada node Westalis pada file `/etc/default/isc-dhcp-server` yang kita buat konfigurasinya pada file temporary yaitu `isc-dhcp-server-1` dengan ditambahkan `INTERFACES="eth0"`

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
INTERFACES="eth0"
```

Setelah itu, kita jalankan script `soal1.sh` pada node Westalis yang berisi:

```shell
cp /root/isc-dhcp-server-1 /etc/default/isc-dhcp-server
service isc-dhcp-server restart
```
 
Hal ini membuat masing-masing node memiliki server tersendiri.

### 2
> dan Ostania sebagai DHCP Relay
### Penyelesaian
Kita dapat melakukan penginstallan di node Ostania yang dilakukan langsung di file `/root/.bashrc` dengan format sebagai berikut:

```shell
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.16.0.0/16
cat /etc/resolv.conf
apt-get update
echo "" | apt-get install isc-dhcp-relay -y
```

Setelah itu, lakukan konfigurasi pada node Ostania pada file `/etc/default/isc-dhcp-relay` yang kita buat konfigurasinya pada file temporary yaitu `isc-dhcp-relay-2` yang berisi:

```shell
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts
#
# This is a POSIX shell fragment
#
# What servers should the DHCP relay forward requests to?
SERVERS="10.16.2.4"
# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"
# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```

Lalu, kita jalankan script `soal2.sh` pada node Ostania yang berisi:

```shell
cp /root/isc-dhcp-relay-2 /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```

Hal ini membuat node Ostania menjadi DHCP relay yang Servernya mengarah ke Westalis serta Interfacenya mengarah ke eth1, eth2, dan eth3 yang merupakan switch 1, 2, dan 3.

> Loid dan Franky menyusun peta tersebut dengan hati-hati dan teliti. Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu:
### 3-6
> Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server
### Penyelesaian
Untuk membuat semua client menggunakan konfigurasi IP dari DHCP Server, maka kita harus edit network configuration untuk masing-masing node menjadi:

```shell
auto eth0
iface eth0 inet dhcp
```

Setelah itu, kita lakukan konfigurasi pada file `/etc/dhcp/dhcpd.conf` pada node Westalis untuk mengatur parameter jaringan yang dapat didistribusikan oleh DHCP. Konfigurasi ini akan dihasilkan dengan file penuh `dhcpd3-6.conf` dengan script sebagai berikut

```shell
#
# Sample configuration file for ISC dhcpd for Debian
#
# Attention: If /etc/ltsp/dhcpd.conf exists, that will be used as
# configuration file instead of this file.
#
#
subnet 10.16.1.0 netmask 255.255.255.0 {
    range 10.16.1.50 10.16.1.88;
    range 10.16.1.120 10.16.1.155;
    option routers 10.16.1.1;
    option broadcast-address 10.16.1.255;
    option domain-name-servers 10.16.2.2; 
    default-lease-time 300;
    max-lease-time 6900;
}
subnet 10.16.3.0 netmask 255.255.255.0 {
    range 10.16.3.10 10.16.3.30;
    range 10.16.3.60 10.16.3.85;
    option routers 10.16.3.1;
    option broadcast-address 10.16.3.255;
    option domain-name-servers 10.16.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}
subnet 10.16.2.0 netmask 255.255.255.0 {
    option routers 10.16.2.1;
}
# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;
# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;
default-lease-time 600;
max-lease-time 7200;
# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;
# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;
# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.
#subnet 10.152.187.0 netmask 255.255.255.0 {
#}
# This is a very basic subnet declaration.
#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}
# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.
#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}
# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option subnet-mask 255.255.255.224;
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}
# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.
#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.fugue.com";
#}
# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
#host fantasia {
#  hardware ethernet 08:00:07:26:c0:a5;
#  fixed-address fantasia.fugue.com;
#}
# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.
#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}
#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
#}
```

Lalu, akan menjalankan script `soal3-6.sh` dengan isi sebagai berikut

```shell
cp /root/dhcpd3-6.conf /etc/dhcp/dhcpd.conf
service isc-dhcp-server restart
```

### 3
> Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155
### Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet `10.16.1.0` yang dilalui Switch1 dengan melakukan penetapan range IP sebagai berikut

```shell
subnet 10.16.1.0 netmask 255.255.255.0 {
    range 10.16.1.50 10.16.1.88;
    range 10.16.1.120 10.16.1.155;
    ...
}
subnet 10.16.3.0 netmask 255.255.255.0 {
    ...
}
subnet 10.16.2.0 netmask 255.255.255.0 {
    ...
}
```

Hal ini akan membuat Client yang melalui Switch1 memiliki range IP yang sudah ditetapkan.

### 4
> Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85
### Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet `10.16.3.0` yang dilalui Switch3 dengan melakukan penetapan range IP sebagai berikut

```shell
subnet 10.16.1.0 netmask 255.255.255.0 {
    ...
}
subnet 10.16.3.0 netmask 255.255.255.0 {
    range 10.16.3.10 10.16.3.30;
    range 10.16.3.60 10.16.3.85;
    ...
}
subnet 10.16.2.0 netmask 255.255.255.0 {
    ...
}
```

Hal ini akan membuat Client yang melalui Switch3 memiliki range IP yang sudah ditetapkan.

### 5
> Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut
### Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet yang dilalui Switch1 dan Switch3 dengan melakukan penetapan IP DNS Server yang terhubung sebagai berikut

```shell
subnet 10.16.1.0 netmask 255.255.255.0 {
    ...
    option domain-name-servers 10.16.2.2; 
    ...
}
subnet 10.16.3.0 netmask 255.255.255.0 {
    ...
    option domain-name-servers 10.16.2.2; 
    ...
}
subnet 10.16.2.0 netmask 255.255.255.0 {
    ...
}
```

Selain itu, kita juga akan membuat suatu temporary file untuk mengedit konfigurasi file `/etc/bind/named.conf.options` yaitu pada file `named-5.conf.options` dengan isi sebagai berikut

```shell
options {
        directory "/var/cache/bind";
        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113
        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.
        forwarders {
            192.168.122.1;
        };
        //=====================================================================$
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //=====================================================================$
        //dnssec-validation auto;
        allow-query{any;};
        
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
```
Lalu, akan menjalankan script `soal5.sh` dengan isi sebagai berikut
```shell
cp /root/named-5.conf.options /etc/bind/named.conf.options
service bind9 restart
```
Hal ini akan membuat Client mendapatkan DNS dari WISE dan terhubung dengan internet melalui DNS.
### 6
> Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit
### Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet yang dilalui Switch1 dan Switch3 dengan melakukan penetapan lama waktu dan waktu maksimal untuk peminjaman alamat IP kepada Client oleh DHCP Server sebagai berikut
```shell
subnet 10.16.1.0 netmask 255.255.255.0 {
    ...
    default-lease-time 300;
    max-lease-time 6900;
}
subnet 10.16.3.0 netmask 255.255.255.0 {
    ...
    default-lease-time 600;
    max-lease-time 6900;
}
subnet 10.16.2.0 netmask 255.255.255.0 {
    ...
}
```
Hal ini akan membuat lama waktu untuk peminjaman alamat IP kepada Client oleh DHCP Server melalui Switch1 selama 5menit (300detik) dan melalui Switch3 selama 10menit (600detik), serta waktu maksimal peminjamannya selama 115menit (6900detik).
### 7
> Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13
### Penyelesaian
Pertama, kita akan mendapatkan hwaddress pada node Eden dengan menggunakan command `ip a` dan akan didapatkan nilai `4a:99:b7:2c:f9:c2` pada `link/ether`
Setelah itu, dapat dibuat file temporary `dhcpd7.conf` dengan isi sebagai berikut
```shell
#
# Sample configuration file for ISC dhcpd for Debian
#
# Attention: If /etc/ltsp/dhcpd.conf exists, that will be used as
# configuration file instead of this file.
#
#
subnet 10.16.1.0 netmask 255.255.255.0 {
    range 10.16.1.50 10.16.1.88;
    range 10.16.1.120 10.16.1.155;
    option routers 10.16.1.1;
    option broadcast-address 10.16.1.255;
    option domain-name-servers 10.16.2.2; 
    default-lease-time 300;
    max-lease-time 6900;
}
subnet 10.16.3.0 netmask 255.255.255.0 {
    range 10.16.3.10 10.16.3.30;
    range 10.16.3.60 10.16.3.85;
    option routers 10.16.3.1;
    option broadcast-address 10.16.3.255;
    option domain-name-servers 10.16.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}
subnet 10.16.2.0 netmask 255.255.255.0 {
    option routers 10.16.2.1;
}
host Eden {
    hardware ethernet f6:18:ab:b8:88:1d;
    fixed-address 10.16.3.13;
}
# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;
# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;
default-lease-time 600;
max-lease-time 7200;
# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;
# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;
# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.
#subnet 10.152.187.0 netmask 255.255.255.0 {
#}
# This is a very basic subnet declaration.
#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}
# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.
#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}
# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option subnet-mask 255.255.255.224;
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}
# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.
#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.fugue.com";
#}
# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
#host fantasia {
#  hardware ethernet 08:00:07:26:c0:a5;
#  fixed-address fantasia.fugue.com;
#}
# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.
#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}
#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
```
Lalu, edit network configuration pada node Eden dengan menambahkan hwaddress sebagai berikut
```shell
auto eth0
iface eth0 inet dhcp
hwaddress ether f6:18:ab:b8:88:1d
```
Setelah itu, jalankan script `soal7.sh` dengan isi sebagai berikut
```shell
cp /root/dhcpd7.conf /etc/dhcp/dhcpd.conf
service isc-dhcp-server restart
```
Hal itu menyebabkan IP fixed address pada node Eden yang memiliki interface eth0 berubah menjadi `10.16.3.13/24`

## Proxy Server
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
SSS, Garden, dan Eden digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin keamanannya, juga untuk mencegah kebocoran data.
Untuk menggunakan Proxy diperlukan untuk mengkonfigurasi proxy terlebih dahulu

### Konfigurasi Proxy
#### Berlint berfungsi sebagai proxy server
Pertama kita membuat backup file config dengan
```
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
```
Pada file /etc/squid/squid.conf tambah kan
```
http_port 8080
visible_hostname Berlint
```
Lalu restart dengan
```
service squid restart
```

#### Pembatasan Waktu Akses
Client hanya dapat mengakses internet di luar hari dan jam kerja (senin-jumat 08:00 - 17:00) dan hari libur (dapat mengakses 24 jam penuh)

Tambahkan file acl.conf
```
nano /etc/squid/acl.conf
```
lalu tambah kan
```
acl UNAVAILABLE_WORKING time MTWHF 08:00-17:00
```

Pada squid.conf tambahkan
```
include /etc/squid/acl.conf
http_port 8080
http_access deny UNAVAILABLE_WORKING
http_access allow all
visible_hostname Berlint
```
Lalu restart squid

#### Pembatasan Akses Website
client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan). Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)

##### Pada WISE
ubah config dengan
```
nano /etc/bind/named.conf.local
```
Lalu tambahkan
```
zone "loid-work.com" {
  type master;
  file "/etc/bind/jarkom/loid-work.com";
};

zone "franky-work.com" {
  type master;
  file "/etc/bind/jarkom/franky-work.com";
};
```

Setelah itu, buat folder baru bernama "jarkom" pada bind
```
mkdir /etc/bind/jarkom/  
```
buat file 
```
nano /etc/bind/jarkom/loid-work.com
```

Lalu Tambahkan 
```
$TTL 604800
@ IN SOA loid-work.com. root.loid-work.com. (
                20211108 ; Serial
                604800 ; Refresh
                86400 ; Retry
                2419200 ; Expire
                604800 ) ; Negative Cache TTL
;
@ IN NS loid-work.com.
@ IN A 10.16.2.3
@ IN AAAA ::1
www IN CNAME loid-work.com.
```

Buat file lagi untuk franky-work.com
```
nano /etc/bind/jarkom/franky-work.com
```

Tambahkan

```
$TTL 604800
@ IN SOA franky-work.com. root.franky-work.com. (
                20211108 ; Serial
                604800 ; Refresh
                86400 ; Retry
                2419200 ; Expire
                604800 ) ; Negative Cache TTL
;
@ IN NS franky-work.com.
@ IN A 10.16.2.3
@ IN AAAA ::1
www IN CNAME franky-work.com.
```

Usai menambahkan file diatas, kita tambahkan konfigurasi squid pada Berlint
dengan menambahkan
```
http_port 5000
visible_hostname loid-work.com
visible_hostname franky-work.com
http_access allow all
```

Lalu restart squid service

#### Pembatasan Bandwith
Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, 
ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)

Membuat file baru pada Berlint dengan
```
nano /etc/squid/acl-bandwidth.conf
```
lalu masukan
```
delay_pools 2
delay_class 1 1
delay_class 1 2
delay_access 1 allow all
delay_access 2 allow all
delay_parameters 2 128000/128000 128000/128000
```
Setelah itu tambahkan pada squid config
```
include /etc/squid/acl-bandwidth.conf
```
hasil akhir config squid pada berlint berupa

![image](https://user-images.githubusercontent.com/82019030/201510001-5c22325f-3227-433a-b9ed-0b0b73a70a34.png)

Lalu dapat mengecek proxy server melalui SSS, Garden, dan Eden dengan
```
export http_proxy="http://10.16.2.3:8080"
```

serta env | grep -i proxy
![image](https://user-images.githubusercontent.com/82019030/201510078-bd409298-fb25-4eff-8727-bb3469215dfe.png)











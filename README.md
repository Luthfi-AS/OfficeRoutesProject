# Panduan Konfigurasi Jaringan Perusahaan di Cisco Packet Tracer 

## Fase 1: Persiapan dan Penempatan Perangkat

### 1.1 Penempatan Perangkat di Workspace
1. **Buka Cisco Packet Tracer**
2. **Tambahkan Router:**
   - 4x Router 2911 (untuk R1, R2, R3, R4)
   - Letakkan R4 di tengah sebagai Core Router
   - R1, R2, R3 di sekitar R4 sesuai layout lantai
   - Pada R4, tambahkan modul HWIC-4ESW

3. **Tambahkan Switch:**
   - 3x Switch 2960 (Switch1, Switch2, Switch3)
   - Letakkan sesuai dengan lantai masing-masing

4. **Tambahkan Server:**
   - 2x Server (DNS Server, Web Server)
   - Letakkan di area lantai 4 bersama R4

5. **Tambahkan PC:**
   - 35x PC (5 PC untuk setiap ruangan: Adm, IT, Fin, Mkt, Sales, Prod, QC, R&D)
   - Kelompokkan sesuai ruangan dan lantai

### 1.2 Skema IP Address yang Diperbaiki
```
Lantai 1: 192.168.1.0/26 (192.168.1.0 - 192.168.1.63)
Lantai 2: 192.168.2.0/28 (192.168.2.0 - 192.168.2.15)
Lantai 3: 192.168.3.0/26 (192.168.3.0 - 192.168.3.63)
Server  : 192.168.10.96/29 (192.168.10.96 - 192.168.10.103)

Inter-Router Links:
R4-R1: 10.0.1.0/30
R4-R2: 10.0.2.0/30
R4-R3: 10.0.3.0/30
R4-Server: 10.0.4.0/30
```

### 1.3 Koneksi Fisik
1. **Koneksi Router ke Router (menggunakan Serial atau Gigabit):**
   - R4 G0/0 ↔ R1 G0/0
   - R4 G0/1 ↔ R2 G0/0  
   - R4 G0/2 ↔ R3 G0/0
   - R4 S0/3/0 ↔ Server-SW (gunakan DCE cable, set clock rate)

2. **Koneksi Router ke Switch:**
   - R1 G0/1 ↔ Switch1 G0/1
   - R2 G0/1 ↔ Switch2 G0/1
   - R3 G0/1 ↔ Switch3 G0/1

3. **Koneksi Server ke Server-SW:**
   - DNS Server NIC ↔ Server-SW F0/1
   - Web Server NIC ↔ Server-SW F0/2

4. **Koneksi PC ke Switch (gunakan Straight-Through Cable):**
   - **Switch1:** PC Adm (F0/2-F0/6), PC IT (F0/7-F0/11), PC Fin (F0/12-F0/16)
   - **Switch2:** PC Mkt (F0/2-F0/6), PC Sales (F0/7-F0/11)
   - **Switch3:** PC Prod (F0/2-F0/6), PC QC (F0/7-F0/11), PC R&D (F0/12-F0/16)

## Fase 2: Konfigurasi Dasar Router

### 2.1 Konfigurasi Router R4 (Core Router)
```
Router> enable
Router# configure terminal
Router(config)# hostname R4
R4(config)# 

! Konfigurasi Interface G0/0 (ke R1)
R4(config)# interface gigabitEthernet 0/0
R4(config-if)# ip address 10.0.1.1 255.255.255.252
R4(config-if)# ipv6 address fd10:0:1::1/64
R4(config-if)# no shutdown
R4(config-if)# exit

! Konfigurasi Interface G0/1 (ke R2)
R4(config)# interface gigabitEthernet 0/1
R4(config-if)# ip address 10.0.2.1 255.255.255.252
R4(config-if)# ipv6 address fd10:0:2::1/64
R4(config-if)# no shutdown
R4(config-if)# exit

! Konfigurasi Interface G0/2 (ke R3)
R4(config)# interface gigabitEthernet 0/2
R4(config-if)# ip address 10.0.3.1 255.255.255.252
R4(config-if)# ipv6 address fd10:0:3::1/64
R4(config-if)# no shutdown
R4(config-if)# exit

! BARU: Konfigurasi Interface FastEthernet untuk Server (dari HWIC-4ESW)
! Konfigurasi port untuk DNS Server
R4(config)# interface fastEthernet 0/1/0
R4(config-if)# switchport mode access
R4(config-if)# switchport access vlan 10
R4(config-if)# no shutdown
R4(config-if)# exit

! Konfigurasi port untuk Web Server
R4(config)# interface fastEthernet 0/1/1
R4(config-if)# switchport mode access
R4(config-if)# switchport access vlan 10
R4(config-if)# no shutdown
R4(config-if)# exit

! Konfigurasi VLAN Interface untuk Server (VLAN akan otomatis terbuat)
R4(config)# interface vlan 10
R4(config-if)# ip address 192.168.10.97 255.255.255.248
R4(config-if)# ipv6 address fd10:0:4::1/64
R4(config-if)# no shutdown
R4(config-if)# exit

! Static routes ke semua subnet (tetap sama)
R4(config)# ip route 192.168.1.0 255.255.255.192 10.0.1.2
R4(config)# ip route 192.168.2.0 255.255.255.240 10.0.2.2
R4(config)# ip route 192.168.3.0 255.255.255.192 10.0.3.2

! Enable IPv6 routing
R4(config)# ipv6 unicast-routing
R4(config)# ipv6 route fd10:0:1:1::/64 fd10:0:1::2
R4(config)# ipv6 route fd10:0:2:1::/64 fd10:0:2::2
R4(config)# ipv6 route fd10:0:3:1::/64 fd10:0:3::2

! DHCP Pool untuk Server (jika diperlukan static assignment)
R4(config)# ip dhcp pool SERVER-POOL
R4(dhcp-config)# network 192.168.10.96 255.255.255.248
R4(dhcp-config)# default-router 192.168.10.97
R4(dhcp-config)# dns-server 192.168.10.98
R4(dhcp-config)# exit

! Exclude server addresses (karena menggunakan static)
R4(config)# ip dhcp excluded-address 192.168.10.96 192.168.10.103

R4(config)# exit
R4# write memory
```

### 2.2 Konfigurasi Router R1 (Lantai 1)
```
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)#

! Konfigurasi Interface G0/0 (ke R4)
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip address 10.0.1.2 255.255.255.252
R1(config-if)# ipv6 address fd10:0:1::2/64
R1(config-if)# no shutdown
R1(config-if)# exit

! Konfigurasi Interface G0/1 (ke Switch1)
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ip address 192.168.1.1 255.255.255.192
R1(config-if)# ipv6 address fd10:0:1:1::1/64
R1(config-if)# no shutdown
R1(config-if)# exit

! Konfigurasi DHCP Pool untuk Lantai 1
R1(config)# ip dhcp pool LANTAI1
R1(dhcp-config)# network 192.168.1.0 255.255.255.192
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# dns-server 192.168.10.98
R1(dhcp-config)# exit

! Exclude addresses
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.5

! Enable IPv6 routing dan SLAAC
R1(config)# ipv6 unicast-routing
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ipv6 enable
R1(config-if)# exit

! Default route
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.1.1
R1(config)# ipv6 route ::/0 fd10:0:1::1
R1(config)# exit
R1# write memory
```

### 2.3 Konfigurasi Router R2 (Lantai 2)
```
Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)#

! Konfigurasi Interface G0/0 (ke R4)
R2(config)# interface gigabitEthernet 0/0
R2(config-if)# ip address 10.0.2.2 255.255.255.252
R2(config-if)# ipv6 address fd10:0:2::2/64
R2(config-if)# no shutdown
R2(config-if)# exit

! Konfigurasi Interface G0/1 (ke Switch2)
R2(config)# interface gigabitEthernet 0/1
R2(config-if)# ip address 192.168.2.1 255.255.255.240
R2(config-if)# ipv6 address fd10:0:2:1::1/64
R2(config-if)# no shutdown
R2(config-if)# exit

! Konfigurasi DHCP Pool untuk Lantai 2
R2(config)# ip dhcp pool LANTAI2
R2(dhcp-config)# network 192.168.2.0 255.255.255.240
R2(dhcp-config)# default-router 192.168.2.1
R2(dhcp-config)# dns-server 192.168.10.98
R2(dhcp-config)# exit

! Exclude addresses
R2(config)# ip dhcp excluded-address 192.168.2.1 192.168.2.3

! Enable IPv6 routing dan SLAAC
R2(config)# ipv6 unicast-routing
R2(config)# interface gigabitEthernet 0/1
R2(config-if)# ipv6 enable
R2(config-if)# exit

! Default route
R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.2.1
R2(config)# ipv6 route ::/0 fd10:0:2::1
R2(config)# exit
R2# write memory
```

### 2.4 Konfigurasi Router R3 (Lantai 3)
```
Router> enable
Router# configure terminal
Router(config)# hostname R3
R3(config)#

! Konfigurasi Interface G0/0 (ke R4)
R3(config)# interface gigabitEthernet 0/0
R3(config-if)# ip address 10.0.3.2 255.255.255.252
R3(config-if)# ipv6 address fd10:0:3::2/64
R3(config-if)# no shutdown
R3(config-if)# exit

! Konfigurasi Interface G0/1 (ke Switch3)
R3(config)# interface gigabitEthernet 0/1
R3(config-if)# ip address 192.168.3.1 255.255.255.192
R3(config-if)# ipv6 address fd10:0:3:1::1/64
R3(config-if)# no shutdown
R3(config-if)# exit

! Konfigurasi DHCP Pool untuk Lantai 3
R3(config)# ip dhcp pool LANTAI3
R3(dhcp-config)# network 192.168.3.0 255.255.255.192
R3(dhcp-config)# default-router 192.168.3.1
R3(dhcp-config)# dns-server 192.168.10.98
R3(dhcp-config)# exit

! Exclude addresses
R3(config)# ip dhcp excluded-address 192.168.3.1 192.168.3.5

! Enable IPv6 routing dan SLAAC
R3(config)# ipv6 unicast-routing
R3(config)# interface gigabitEthernet 0/1
R3(config-if)# ipv6 enable
R3(config-if)# exit

! Default route
R3(config)# ip route 0.0.0.0 0.0.0.0 10.0.3.1
R3(config)# ipv6 route ::/0 fd10:0:3::1
R3(config)# exit
R3# write memory
```

## Fase 3: Konfigurasi Switch dan VLAN

### 3.1 Konfigurasi Switch1 (Lantai 1)
```
Switch> enable
Switch# configure terminal
Switch(config)# hostname Switch1
Switch1(config)#


! Set default gateway
Switch1(config)# ip default-gateway 192.168.1.1

! Konfigurasi port untuk PC (access mode)
Switch1(config)# interface range fastEthernet 0/1-24
Switch1(config-if-range)# switchport mode access
Switch1(config-if-range)# switchport access vlan 1
Switch1(config-if-range)# exit

! Konfigurasi uplink ke router (access mode)
Switch1(config)# interface gigabitEthernet 0/1
Switch1(config-if)# switchport mode access
Switch1(config-if)# exit

! Enable SSH untuk management
Switch1(config)# ip domain-name perusahaan.local
Switch1(config)# crypto key generate rsa
Switch1(config)# username admin privilege 15 secret admin123
Switch1(config)# line vty 0 15
Switch1(config-line)# login local
Switch1(config-line)# transport input telnet
Switch1(config-line)# exit
Switch1(config)# exit

Switch1# configure terminal
Switch1(config)# interface vlan 1
Switch1(config-if)# ip address 192.168.1.2 255.255.255.192
Switch1(config-if)# no shutdown
Switch1(config-if)# exit
Switch1(config)# ip default-gateway 192.168.1.1

Switch1(config)# interface range fastEthernet 0/2-16
Switch1(config-if-range)# duplex full
Switch1(config-if-range)# speed 100
Switch1(config-if-range)# exit

! Aktifkan Auto-MDIX untuk semua port FastEthernet
Switch1(config)# interface range fastEthernet 0/1-24
Switch1(config-if-range)# mdix auto
Switch1(config-if-range)# exit

Switch1(config)# interface gigabitEthernet 0/1
Switch1(config-if)# duplex full
Switch1(config-if)# speed 1000
Switch1(config-if)# exit

! Aktifkan Auto-MDIX untuk port GigabitEthernet
Switch1(config)# interface gigabitEthernet 0/1
Switch1(config-if)# mdix auto
Switch1(config-if)# exit

Switch1(config)# interface range fastEthernet 0/1-24
Switch1(config-if-range)# mdix auto
Switch1(config-if-range)# exit
Switch1(config)# exit

Switch1# write memory
```

### 3.2 Konfigurasi Switch2 (Lantai 2)
```
Switch> enable
Switch# configure terminal
Switch(config)# hostname Switch2
Switch2(config)#

! Set default gateway
Switch2(config)# ip default-gateway 192.168.2.1

! Konfigurasi port untuk PC
Switch2(config)# interface range fastEthernet 0/1-24
Switch2(config-if-range)# switchport mode access
Switch2(config-if-range)# switchport access vlan 1
Switch2(config-if-range)# exit

! Konfigurasi uplink ke router
Switch2(config)# interface gigabitEthernet 0/1
Switch2(config-if)# switchport mode access
Switch2(config-if)# exit

! Enable SSH untuk management
Switch2(config)# ip domain-name perusahaan.local
Switch2(config)# crypto key generate rsa
Switch2(config)# username admin privilege 15 secret admin123
Switch2(config)# line vty 0 15
Switch2(config-line)# login local
Switch2(config-line)# transport input telnet
Switch2(config-line)# exit
Switch2(config)# exit

Switch2# configure terminal
Switch2(config)# interface vlan 1
Switch2(config-if)# ip address 192.168.2.2 255.255.255.240
Switch2(config-if)# no shutdown
Switch2(config-if)# exit
Switch2(config)# ip default-gateway 192.168.2.1

Switch2(config)# interface range fastEthernet 0/3-24
Switch2(config-if-range)# mdix auto
Switch2(config-if-range)# exit

! Konfigurasi port untuk PC (hanya F0/2-11 yang digunakan)
Switch2(config)# interface range fastEthernet 0/2-11
Switch2(config-if-range)# duplex full
Switch2(config-if-range)# speed 100
Switch2(config-if-range)# exit

! Konfigurasi uplink ke router
Switch2(config)# interface gigabitEthernet 0/1
Switch2(config-if)# duplex full
Switch2(config-if)# speed 1000
Switch2(config-if)# exit
Switch2(config)# exit
Switch2# write memory
```

### 3.3 Konfigurasi Switch3 (Lantai 3)
```
Switch> enable
Switch# configure terminal
Switch(config)# hostname Switch3
Switch3(config)#

! Set default gateway
Switch3(config)# ip default-gateway 192.168.3.1

! Konfigurasi port untuk PC
Switch3(config)# interface range fastEthernet 0/1-24
Switch3(config-if-range)# switchport mode access
Switch3(config-if-range)# switchport access vlan 1
Switch3(config-if-range)# exit

! Konfigurasi uplink ke router
Switch3(config)# interface gigabitEthernet 0/1
Switch3(config-if)# switchport mode access
Switch3(config-if)# exit

! Enable SSH untuk management
Switch3(config)# ip domain-name perusahaan.local
Switch3(config)# crypto key generate rsa
Switch3(config)# username admin privilege 15 secret admin123
Switch3(config)# line vty 0 15
Switch3(config-line)# login local
Switch3(config-line)# transport input telnet
Switch3(config-line)# exit
Switch3(config)# exit

Switch3# configure terminal
Switch3(config)# interface vlan 1
Switch3(config-if)# ip address 192.168.3.2 255.255.255.192
Switch3(config-if)# no shutdown
Switch3(config-if)# exit
Switch3(config)# ip default-gateway 192.168.3.1

Switch3(config)# interface range fastEthernet 0/1-24
Switch3(config-if-range)# mdix auto
Switch3(config-if-range)# exit
Switch3(config)# exit

! Konfigurasi port untuk PC (F0/2-16 yang digunakan)
Switch3(config)# interface range fastEthernet 0/2-16
Switch3(config-if-range)# duplex full
Switch3(config-if-range)# speed 100
Switch3(config-if-range)# exit

! Konfigurasi uplink ke router
Switch3(config)# interface gigabitEthernet 0/1
Switch3(config-if)# duplex full
Switch3(config-if)# speed 1000
Switch3(config-if)# exit

Switch3# write memory
```

## Fase 4: Konfigurasi Server

### 4.1 Konfigurasi DNS Server
1. **Klik DNS Server → Desktop → IP Configuration**
   - IPv4 Address: `192.168.10.98`
   - Subnet Mask: `255.255.255.248`
   - Default Gateway: F
   - DNS Server: `192.168.10.98` (dirinya sendiri)

2. **IPv6 Configuration:**
   - IPv6 Address: `fd10:0:4::98/64`
   - IPv6 Gateway: `fd10:0:4::1`

3. **Aktifkan DNS Service:**
   - Klik **Services → DNS**
   - Turn ON DNS Service
   - Tambahkan record:
     - `perusahaan.local` → `192.168.10.99`
     - `www.perusahaan.local` → `192.168.10.99`
     - `dns.perusahaan.local` → `192.168.10.98`

### 4.2 Konfigurasi Web Server
1. **Klik Web Server → Desktop → IP Configuration**
   - IPv4 Address: `192.168.10.99`
   - Subnet Mask: `255.255.255.248`
   - Default Gateway: `192.168.10.97`
   - DNS Server: `192.168.10.98`

2. **IPv6 Configuration:**
   - IPv6 Address: `fd10:0:4::99/64`
   - IPv6 Gateway: `fd10:0:4::1`

3. **Aktifkan HTTP Service:**
   - Klik **Services → HTTP**
   - Turn ON HTTP Service
   - Edit index.html sesuai kebutuhan perusahaan

## Fase 5: Konfigurasi PC (DHCP Client)

### 5.1 Konfigurasi Semua PC
Untuk setiap PC di semua lantai:

1. **Klik PC → Desktop → IP Configuration**
2. **Pilih DHCP untuk IPv4**
3. **Pilih Auto Config untuk IPv6**
4. **Verifikasi IP yang diperoleh sesuai dengan range yang ditentukan**

## Fase 6: Testing dan Verifikasi

### 6.1 Test Konektivitas Dasar
1. **Ping Test antar lantai:**
   ```
   PC> ping 192.168.10.98 (DNS Server)
   PC> ping 192.168.10.99 (Web Server)
   PC> ping [IP PC di lantai lain]
   ```

2. **Test DNS Resolution:**
   ```
   PC> nslookup perusahaan.local
   PC> nslookup www.perusahaan.local
   ```

3. **Test Web Access:**
   - Buka Web Browser di PC
   - Akses `http://perusahaan.local` atau `http://192.168.10.99`

### 6.2 Test Management Access
1. **SSH ke Switch dari PC IT Support:**
   ```
   PC> telnet 192.168.1.2 (Switch1)
   PC> telnet 192.168.2.2 (Switch2)
   PC> telnet 192.168.3.2 (Switch3)
   ```

### 6.3 Verifikasi Routing
1. **Di R4, cek routing table:**
   ```
   R4# show ip route
   R4# show ipv6 route
   ```

2. **Test traceroute:**
   ```
   PC> tracert 192.168.10.99
   ```








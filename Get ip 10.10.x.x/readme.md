# MikroTik LAN Setup: IP 10.10.10.x

Panduan konfigurasi jaringan lokal (LAN) menggunakan MikroTik dengan segmen IP privat `10.10.10.0/24`, mencakup bridge, DHCP server, static lease, dan NAT.

---

## Daftar Isi

1. [Memahami IP 10.10.10.x](#1-memahami-ip-10101x)
2. [Membuat Bridge LAN](#2-membuat-bridge-lan)
3. [Memberikan IP Gateway ke Bridge](#3-memberikan-ip-gateway-ke-bridge)
4. [Setup DHCP Server](#4-setup-dhcp-server)
5. [Membuat IP Statis via DHCP Lease](#5-membuat-ip-statis-via-dhcp-lease)
6. [Mengaktifkan Akses Internet dengan NAT](#6-mengaktifkan-akses-internet-dengan-nat)

---

## 1. Memahami IP 10.10.10.x

`10.0.0.0/8` adalah blok IP privat Kelas A. Alasan segmen ini umum dipakai di lingkungan profesional:

- **Skalabilitas**: Blok `10.x.x.x` secara teori menampung hingga 16 juta alamat host, jauh lebih luas dibanding `192.168.x.x` (Kelas C) yang terbatas sekitar 65 ribu. Ruang yang besar mengurangi risiko konflik saat jaringan berkembang.
- **Efisiensi**: Pengetikan `10.10.10.x` lebih singkat dibanding `192.168.100.x`, relevan untuk admin yang sering bekerja lewat CLI.
- **Konvensi**: Penggunaan `10.x.x.x` umum ditemukan di infrastruktur data center dan enterprise, sehingga mencerminkan pengelolaan jaringan yang terstruktur.

---

## 2. Membuat Bridge LAN

Bridge menyatukan beberapa port fisik MikroTik menjadi satu segmen jaringan, sehingga perangkat yang terhubung ke port berbeda tetap bisa saling berkomunikasi.

**Langkah-langkah (via Winbox):**

1. Buka menu **Bridge**.
2. Klik tombol **+**, beri nama `bridge-LAN`, lalu klik **OK**.
3. Masuk ke tab **Ports**, klik **+**.
4. Pilih interface yang akan digabungkan (contoh: `ether2`, `ether3`, dst.).
5. Pilih Bridge: `bridge-LAN`, lalu klik **OK**.
6. Ulangi untuk setiap port yang ingin dimasukkan.

> Jangan masukkan port yang terhubung ke modem/internet (biasanya `ether1`) ke dalam bridge ini.

---

## 3. Memberikan IP Gateway ke Bridge

Setelah port digabungkan, berikan alamat IP ke `bridge-LAN`. Alamat ini akan menjadi gateway bagi semua perangkat di jaringan lokal.

**Langkah-langkah:**

1. Buka menu **IP > Addresses**.
2. Klik **+**, isi:
   - Address: `10.10.10.1/24`
   - Interface: `bridge-LAN`
3. Klik **OK**.

> Notasi `/24` berarti subnet mask `255.255.255.0`, menyediakan 254 alamat host yang dapat digunakan (`.1` sampai `.254`). Alamat `.1` digunakan untuk router sebagai gateway.

---

## 4. Setup DHCP Server

DHCP server bertugas memberikan alamat IP secara otomatis kepada setiap perangkat yang terhubung ke jaringan, tanpa perlu konfigurasi manual di sisi klien.

**Langkah-langkah:**

1. Buka menu **IP > DHCP Server**.
2. Klik tombol **DHCP Setup** (bukan tombol **+**).
3. Pilih interface: `bridge-LAN`, klik **Next**.
4. DHCP Address Space: pastikan terisi `10.10.10.0/24`, klik **Next**.
5. Gateway for DHCP Network: pastikan terisi `10.10.10.1`, klik **Next**.
6. Addresses to Give Out: tentukan rentang IP yang akan dibagikan, contoh `10.10.10.10-10.10.10.254`, klik **Next**.
7. DNS Servers: isi dengan `8.8.8.8` (Google DNS) atau `10.10.10.1` jika router juga menjalankan DNS lokal, klik **Next**.
8. Klik **Next** hingga proses selesai.

> Rentang dimulai dari `.10` untuk menyisakan `.2` sampai `.9` sebagai cadangan bagi perangkat dengan IP statis yang dikonfigurasi manual.

---

## 5. Membuat IP Statis via DHCP Lease

Fitur ini mengunci perangkat tertentu agar selalu mendapatkan IP yang sama setiap kali terhubung, berdasarkan MAC address-nya. Berguna untuk server, NAS, atau perangkat lain yang membutuhkan alamat tetap.

**Langkah-langkah:**

1. Buka menu **IP > DHCP Server**, lalu masuk ke tab **Leases**.
2. Tunggu hingga perangkat target terhubung dan muncul di daftar.
3. Klik kanan pada perangkat tersebut, pilih **Make Static**.
4. Untuk mengubah alamat IP-nya, klik dua kali pada baris tersebut dan ganti nilai di kolom **Address** (contoh: `10.10.10.50`).

**Verifikasi via ARP:**

1. Buka menu **IP > ARP**.
2. Daftar ini menampilkan pasangan IP dan MAC address dari perangkat yang aktif di jaringan.

> ARP (Address Resolution Protocol) memetakan alamat IP ke alamat fisik (MAC address). Tabel ARP berguna untuk memverifikasi perangkat mana yang sedang aktif dan alamat IP mana yang sedang digunakan.

---

## 6. Mengaktifkan Akses Internet dengan NAT

NAT (Network Address Translation) memungkinkan perangkat dengan IP privat (`10.10.10.x`) mengakses internet melalui satu IP publik milik router.

**Langkah-langkah:**

1. Buka menu **IP > Firewall**, masuk ke tab **NAT**.
2. Klik **+**, isi pada tab **General**:
   - Chain: `srcnat`
   - Out Interface: pilih interface yang mengarah ke modem/internet (contoh: `ether1`)
3. Pindah ke tab **Action**, pilih:
   - Action: `masquerade`
4. Klik **OK**.

> `masquerade` secara otomatis mengganti alamat sumber (IP privat) dengan IP publik router saat paket keluar ke internet, lalu mengembalikan respons ke perangkat yang tepat saat paket masuk kembali.

---

## Ringkasan Alur Konfigurasi

```
[Modem/ISP]
     |
  ether1 (WAN)
     |
[MikroTik Router]  <-- IP publik dari ISP
     |
  bridge-LAN (10.10.10.1/24)
     |
  ether2, ether3, dst. (LAN)
     |
[Perangkat klien]  <-- IP otomatis dari DHCP: 10.10.10.10 - 10.10.10.254
```

---

## Referensi

- [MikroTik Wiki - Bridge](https://wiki.mikrotik.com/wiki/Manual:Interface/Bridge)
- [MikroTik Wiki - DHCP Server](https://wiki.mikrotik.com/wiki/Manual:IP/DHCP_Server)
- [MikroTik Wiki - NAT](https://wiki.mikrotik.com/wiki/Manual:IP/Firewall/NAT)
- [RFC 1918 - Address Allocation for Private Internets](https://datatracker.ietf.org/doc/html/rfc1918)

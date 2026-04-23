# MikroTik LAN Setup: IP 10.10.10.x

Panduan konfigurasi jaringan lokal MikroTik dari nol — mulai dari menyatukan port, membagikan IP otomatis, hingga koneksi internet bisa jalan.

---

## Kenapa Pakai IP 10.10.10.x?

Blok `10.x.x.x` adalah IP privat Kelas A yang bisa menampung hingga 16 juta perangkat, jauh lebih luas dibanding `192.168.x.x` yang umum dipakai router rumahan. Di lingkungan profesional, ruang yang besar ini penting agar jaringan tidak kehabisan alamat saat terus berkembang, dan juga ip 10.10.10.x memudahkan dalam memanajemen ip yang sudah ada karena mudah untuk diingat.

---

## 1. Bridge — Menyatukan Port Fisik

Secara default, tiap port di MikroTik berdiri sendiri. Bridge menyatukannya menjadi satu jaringan, sehingga perangkat yang colok di port berbeda tetap bisa saling berkomunikasi.

1. Buka **Bridge**, klik **+**, beri nama `bridge-LAN`, klik **OK**.
2. Masuk tab **Ports**, klik **+**.
3. Pilih interface (contoh: `ether2`, `ether3`), pilih Bridge: `bridge-LAN`, klik **OK**.
4. Ulangi untuk setiap port LAN yang ingin digunakan.

> Jangan masukkan port yang terhubung ke modem (biasanya `ether1`).

---

## 2. IP Address — Memberi Identitas ke Router

Ini adalah alamat gateway yang akan digunakan semua perangkat di jaringan untuk "keluar" melalui router.

1. Buka **IP > Addresses**, klik **+**.
2. Isi Address: `10.10.10.1/24`, pilih Interface: `bridge-LAN`.
3. Klik **OK**.

> `/24` artinya tersedia 254 slot IP untuk perangkat (`.1` sampai `.254`). Alamat `.1` dipakai untuk router.

---

## 3. DHCP Server — Membagi IP Otomatis

Agar perangkat yang terhubung langsung dapat IP tanpa perlu disetting manual.

1. Buka **IP > DHCP Server**, klik **DHCP Setup**.
2. Interface: `bridge-LAN`, klik **Next**.
3. Address Space: `10.10.10.0/24`, klik **Next**.
4. Gateway: `10.10.10.1`, klik **Next**.
5. Rentang IP: `10.10.10.10 - 10.10.10.254`, klik **Next**.
6. DNS: `8.8.8.8`, klik **Next** hingga selesai.

---

## 4. Static Lease — Mengunci IP Perangkat Tertentu

Agar server atau perangkat penting selalu mendapat IP yang sama setiap kali terhubung.

1. Buka **IP > DHCP Server > Leases**.
2. Cari perangkat di daftar, klik kanan, pilih **Make Static**.
3. Klik dua kali pada baris tersebut untuk mengubah alamat IP-nya jika perlu.

Untuk verifikasi, buka **IP > ARP**. Di sini terlihat daftar IP beserta MAC address perangkat yang sedang aktif di jaringan.

---

## 5. NAT — Menghubungkan ke Internet

Perangkat di jaringan lokal tidak bisa langsung akses internet tanpa NAT. Rule ini yang menerjemahkan IP privat ke IP publik milik router saat traffic keluar.

1. Buka **IP > Firewall > NAT**, klik **+**.
2. Tab **General**: Chain `srcnat`, Out Interface: `ether1` (port ke modem).
3. Tab **Action**: pilih `masquerade`.
4. Klik **OK**.

---

## Gambaran Arsitektur

```
[Modem/ISP]
     |
  ether1  <-- WAN (IP Publik)
     |
[MikroTik]
     |
  bridge-LAN (10.10.10.1/24)
     |
  ether2, ether3, dst.
     |
[Perangkat]  <-- DHCP: 10.10.10.10 - 10.10.10.254
```

---

## Referensi

- [MikroTik Wiki - Bridge](https://wiki.mikrotik.com/wiki/Manual:Interface/Bridge)
- [MikroTik Wiki - DHCP Server](https://wiki.mikrotik.com/wiki/Manual:IP/DHCP_Server)
- [MikroTik Wiki - NAT](https://wiki.mikrotik.com/wiki/Manual:IP/Firewall/NAT)
- [RFC 1918 - Private Address Allocation](https://datatracker.ietf.org/doc/html/rfc1918)

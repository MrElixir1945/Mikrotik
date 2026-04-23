# MikroTik DNS over HTTPS (DoH) Setup

DNS biasa itu seperti kartu pos — semua orang di jalurnya bisa membaca isinya, termasuk ISP kamu. DoH (DNS over HTTPS) membungkus permintaan DNS itu di dalam enkripsi HTTPS, sehingga ISP hanya melihat traffic biasa dan tidak bisa tahu situs apa yang kamu akses.

Panduan ini mengkonfigurasi MikroTik agar semua perangkat di jaringan otomatis menggunakan DoH via Cloudflare, termasuk memaksa perangkat yang mencoba kabur ke DNS lain.

---

## 1. Persiapan Sertifikat

DoH berjalan di atas HTTPS. MikroTik perlu sertifikat untuk memverifikasi bahwa server yang dihubungi (Cloudflare) adalah server yang asli, bukan pihak lain yang berpura-pura.

Pada RouterOS 7 ke atas, sertifikat bisa diverifikasi otomatis:

1. Buka **System > Certificates**.
2. Tidak perlu mengimpor manual — MikroTik akan mengambil sertifikat sendiri saat opsi **Verify DoH Certificate** diaktifkan di langkah berikutnya.

> Untuk RouterOS di bawah versi 7, sertifikat Root CA perlu diimpor manual dari DigiCert atau Let's Encrypt.

---

## 2. Konfigurasi DNS ke DoH

Langkah ini mengganti DNS standar dengan jalur DoH Cloudflare.

1. Buka **IP > DNS**.
2. Kosongkan kolom **Servers** dan **Dynamic Servers** — pastikan tidak ada IP di sana.
3. Di kolom **Use DoH Server**, masukkan: `https://cloudflare-dns.com/dns-query`
4. Centang **Verify DoH Certificate** — memastikan koneksi ke Cloudflare terenkripsi dan terverifikasi.
5. Centang **Allow Remote Requests** — agar perangkat di jaringan (HP, laptop) bisa menggunakan DNS dari router ini.

---

## 3. Static DNS — Solusi Ayam dan Telur

Ada satu masalah: MikroTik tidak bisa membuka `cloudflare-dns.com` kalau belum tahu IP-nya, tapi dia baru tahu IP-nya setelah DoH berjalan. Solusinya adalah memberi tahu IP Cloudflare secara manual di awal sebagai titik awal koneksi.

1. Di jendela **IP > DNS**, klik tombol **Static**.
2. Tambahkan entri pertama:
   - Name: `cloudflare-dns.com`
   - Address: `1.1.1.1`
3. Tambahkan entri kedua (cadangan):
   - Name: `cloudflare-dns.com`
   - Address: `1.0.0.1`

---

## 4. NAT Redirect — Memaksa Semua Traffic DNS Lewat Router

Beberapa aplikasi atau perangkat kadang menggunakan server DNS miliknya sendiri, melewati pengaturan router. Rule ini menangkap semua permintaan DNS yang mencoba keluar dan membelokkannya kembali ke MikroTik.

1. Buka **IP > Firewall > NAT**, klik **+**.
2. Buat rule untuk UDP:
   - Chain: `dstnat`
   - Protocol: `udp`
   - Dst. Port: `53`
   - In. Interface: `bridge-LAN`
   - Action: `redirect`
   - To Ports: `53`
3. Ulangi langkah yang sama untuk Protocol: `tcp`.

> Setelah rule ini aktif, ISP hanya akan melihat traffic HTTPS (port 443). Tidak ada lagi DNS query yang terbaca di jaringan provider.

---

## 5. Verifikasi

1. Buka terminal MikroTik, jalankan perintah berikut untuk mengosongkan cache DNS lama:
   ```
   /ip dns cache flush
   ```
2. Dari perangkat yang terhubung ke jaringan, buka [1.1.1.1/help](https://1.1.1.1/help) atau [dnsleaktest.com](https://dnsleaktest.com).
3. Pastikan hasilnya menunjukkan **Using DNS over HTTPS: YES** dan server yang tampil hanya milik Cloudflare.

---

## Gambaran Alur

```
[Perangkat] -- DNS query --> [MikroTik] -- HTTPS (DoH) --> [Cloudflare 1.1.1.1]
                                  |
                          NAT menangkap perangkat
                          yang mencoba bypass DNS
```

---

## Referensi

- [MikroTik Wiki - DNS over HTTPS](https://wiki.mikrotik.com/wiki/DNS_over_HTTPS)
- [Cloudflare DoH](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-https/)
- [RFC 8484 - DNS over HTTPS](https://datatracker.ietf.org/doc/html/rfc8484)

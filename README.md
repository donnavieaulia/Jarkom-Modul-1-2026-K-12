# Laporan Praktikum Modul 1

## Soal 1

**Deskripsi Soal:** Lain berperan sebagai router yang menghubungkan tiga switch. Switch1 menghubungkan Alice dan Mika, Switch2 menghubungkan Chisa, sedangkan Switch3 menghubungkan Knights dan Eiri.

**Media pengujian:** topologi GNS3 dan console setiap node.

**Analisis:** susunan node, hubungan antar-interface, pembagian subnet, dan alamat IP setiap client.

### Penyelesaian dan Hasil Analisis

1. Topologi menggunakan NAT1, router LAIN, tiga switch, dan lima client. Node router/client menggunakan image `ardhptr21/alpinet:latest`.

2. Interface LAIN dihubungkan sebagai berikut:

   | Interface LAIN | Terhubung ke | Jaringan |
   |---|---|---|
   | eth0 | NAT1 | DHCP |
   | eth1 | Switch1 | 192.217.1.0/24 |
   | eth2 | Switch2 | 192.217.2.0/24 |
   | eth3 | Switch3 | 192.217.3.0/24 |

3. Alamat client disesuaikan dengan subnetnya. Setiap client menggunakan `eth0` dan gateway LAIN pada subnet yang sama.

   | Client | Alamat IP | Gateway |
   |---|---|---|
   | Alice | 192.217.1.10/24 | 192.217.1.1 |
   | Mika | 192.217.1.20/24 | 192.217.1.1 |
   | Chisa | 192.217.2.10/24 | 192.217.2.1 |
   | Knights | 192.217.3.10/24 | 192.217.3.1 |
   | Eiri | 192.217.3.20/24 | 192.217.3.1 |

4. Konfigurasi diperiksa pada console masing-masing node menggunakan:

   ```bash
   hostname
   ip -br a
   ip route
   ```

### Hasil

- **Router:** LAIN.
- **Jumlah subnet client:** 3.
- **Jumlah client:** 5.
- **Prefix konfigurasi kelompok:** `192.217`.

### Validasi

Cocokkan alamat `ip -br a` dan gateway `ip route` setiap client dengan tabel konfigurasi.

## Soal 2

**Deskripsi Soal:** LAIN dihubungkan ke internet publik melalui NAT/DHCP pada interface eth0.

**Media pengujian:** console LAIN.

**Analisis:** alamat DHCP pada eth0, default route, serta kemampuan router mengakses internet.

### Penyelesaian dan Hasil Analisis

1. DHCP pada eth0 dijalankan melalui script jaringan saat node dimulai. Perintah yang digunakan untuk meminta lease adalah:

   ```bash
   udhcpc -i eth0 -n -q
   ```

2. Pada output startup LAIN terlihat lease DHCP `192.168.122.240` diperoleh dari server `192.168.122.1`. Alamat ini merupakan hasil sesi tersebut dan dapat berubah pada sesi berikutnya.

3. Alamat dan route diperiksa dari console LAIN:

   ```bash
   ip -4 addr show eth0
   ip route
   ```

4. Koneksi internet diuji dari LAIN:

   ```bash
   ping -c 3 8.8.8.8
   ```

### Hasil

- **Interface internet:** `eth0`.
- **IP DHCP pada sesi yang diamati:** `192.168.122.240/24`.
- **Default gateway pada sesi yang diamati:** `192.168.122.1`.

### Validasi

Jalankan `ping -c 3 8.8.8.8` dari LAIN. Balasan ping menjadi indikator akses internet router.

## Soal 3

**Deskripsi Soal:** Seluruh client pada Switch1, Switch2, dan Switch3 harus dapat saling berkomunikasi melalui router LAIN.

**Media pengujian:** console LAIN dan client.

**Analisis:** IP forwarding, route menuju subnet lain, dan balasan ICMP antar-client.

### Penyelesaian dan Hasil Analisis

1. Pada LAIN, forwarding diaktifkan melalui script jaringan. Nilainya diperiksa dengan:

   ```bash
   sysctl net.ipv4.ip_forward
   ip route
   ```

2. Output LAIN menunjukkan `net.ipv4.ip_forward = 1` dan route langsung ke `192.217.1.0/24`, `192.217.2.0/24`, serta `192.217.3.0/24`. Client menggunakan default gateway subnet masing-masing.

3. Dari console Alice, komunikasi ke seluruh client lain diuji menggunakan:

   ```bash
   ping -c 3 192.217.1.20
   ping -c 3 192.217.2.10
   ping -c 3 192.217.3.10
   ping -c 3 192.217.3.20
   ```

4. Keberhasilan ditentukan dari balasan masing-masing alamat tujuan. Semua subnet client terhubung langsung ke LAIN sehingga route antar-subnet tersedia sebagai connected route.

### Hasil

- **Forwarding LAIN:** aktif (`1`).
- **Route tiga subnet pada router:** tersedia.

### Validasi

Ping dari Alice menuju seluruh client lain harus memperoleh balasan. Forwarding router harus bernilai `1`.

## Soal 4

**Deskripsi Soal:** Client dikonfigurasi agar dapat mengakses internet menggunakan NAT Masquerade pada LAIN dan DNS resolver pada masing-masing client.

**Media pengujian:** console LAIN dan kelima client.

**Analisis:** aturan NAT keluar melalui eth0, resolver DNS, konektivitas ke IP publik, dan akses domain google.com.

### Penyelesaian dan Hasil Analisis

1. Pada LAIN, aturan NAT diperiksa menggunakan:

   ```bash
   iptables -t nat -L -v -n
   ```

2. Output pemeriksaan menunjukkan chain `POSTROUTING` menuju `MODUL1_NAT`. Di chain tersebut tersedia `MASQUERADE` untuk sumber `192.217.0.0/16` keluar melalui `eth0`. Alamat sumber client diterjemahkan ke alamat interface keluar router.

3. Resolver pada setiap client diperiksa menggunakan:

   ```bash
   cat /etc/resolv.conf
   ```

   Konfigurasi yang digunakan berisi nameserver `8.8.8.8` dan `1.1.1.1`.

4. Pada Alice, Mika, Chisa, Knights, dan Eiri, lakukan pengujian yang sama:

   ```bash
   ping -c 3 8.8.8.8
   nslookup google.com
   wget -S -O /dev/null http://google.com
   ```

5. Balasan ping membuktikan akses IP publik. Jawaban DNS membuktikan resolusi domain. Respons HTTP membuktikan akses layanan web; ikuti redirect apabila diberikan oleh server.

### Hasil

- **Aturan NAT:** `MASQUERADE` melalui `eth0`.
- **Sumber jaringan pada aturan:** `192.217.0.0/16`.
- **DNS sesuai konfigurasi:** `8.8.8.8` dan `1.1.1.1`.

### Validasi

Pada setiap client, periksa balasan ping IP publik, jawaban DNS, dan respons HTTP dari google.com.

## Soal 5

**Deskripsi Soal:** Konfigurasi jaringan harus tetap aktif setelah seluruh node di-restart. Script /root/cek_status.sh pada LAIN menampilkan interface dan tabel NAT setelah reboot.

**Media pengujian:** Stop/Start node GNS3 dan console setelah restart.

**Analisis:** pemanggilan script jaringan saat startup, IP interface, default route, NAT, dan forwarding setelah node dinyalakan kembali.

### Penyelesaian dan Hasil Analisis

1. Konfigurasi tiap node disimpan pada `/root/jaringan.sh`. Pada LAIN, script jaringan juga memanggil `/root/nat_modul1.sh` agar aturan NAT dipasang kembali.

2. Pada pengaturan Start command masing-masing node Docker digunakan:

   ```text
   /bin/sh -c 'sh /root/jaringan.sh && exec /bin/bash -i'
   ```

   Nilai ini diisi pada pengaturan GNS3. Nama script sama, tetapi isi konfigurasi setiap node berbeda sesuai perannya.

3. Script verifikasi LAIN memeriksa interface, NAT, route, dan forwarding dengan perintah berikut:

   ```bash
   ip -br a
   iptables -t nat -L -v -n
   ip route
   sysctl net.ipv4.ip_forward
   ```

4. Semua node Docker dihentikan lalu dinyalakan kembali. Tanpa menjalankan script jaringan secara manual, jalankan di LAIN:

   ```bash
   sh /root/cek_status.sh
   ```

5. Pada masing-masing client, lakukan pemeriksaan setelah restart:

   ```bash
   ip -br a
   ip route
   ping -c 3 8.8.8.8
   ```

### Hasil

- **Pada hasil pemeriksaan LAIN:** ketiga IP gateway tersedia, default route tersedia, aturan NAT terpasang, dan forwarding bernilai `1`.
- **Script verifikasi:** `/root/cek_status.sh`.

### Validasi

Setelah Stop/Start seluruh node, jalankan `sh /root/cek_status.sh` pada LAIN dan periksa IP, route, serta ping client tanpa menjalankan setup jaringan manual.

## Soal 6

**Deskripsi Soal:** Mika menjalankan generator traffic untuk mengamati paket DNS dan ICMP menggunakan Wireshark.

**File generator:** `traffic_protocol7.sh`

**File capture:** hasil capture praktikum, disarankan disimpan sebagai `soal06.pcapng`.

**Lokasi capture:** kabel Mika–Switch1.

**Analisis:** jenis paket DNS dan ICMP serta jumlah paket yang lolos display filter.

### Penyelesaian dan Hasil Analisis

1. Klik kanan kabel Mika–Switch1 pada GNS3, pilih **Start capture**, lalu buka Wireshark.

2. Setelah capture aktif, jalankan generator dari console Mika:

   ```bash
   bash /root/traffic_protocol7.sh
   ```

3. Pada Wireshark gunakan display filter:

   ```text
   dns || icmp
   ```

4. Operator `||` menampilkan paket yang memenuhi salah satu kondisi: DNS atau ICMP. DNS digunakan untuk permintaan/respons nama domain; ICMP digunakan untuk Echo Request dan Echo Reply dari ping.

5. Catat jumlah **Displayed**. Gunakan filter `dns` dan `icmp` secara terpisah untuk menghitung rincian paket tiap protokol. Gunakan angka capture aktual, bukan jumlah perkiraan dari script.

### Hasil

- **Protokol yang dianalisis:** DNS dan ICMP.
- **Filter gabungan:** `dns || icmp`.

### Validasi

Filter `dns || icmp` harus menampilkan paket DNS atau ICMP. Jumlah paket dihitung dari nilai Displayed pada capture masing-masing.

## Soal 7

**Deskripsi Soal:** Chisa menyediakan FTP server dengan shared folder /var/wired/data. Alice memiliki akses baca/tulis, Mika hanya membaca, dan Eiri masuk blacklist.

**File capture:** hasil capture praktikum, disarankan `soal07.pcapng`.

**Lokasi capture:** kabel Chisa–Switch2.

**Analisis:** kebijakan akses akun, upload signal_alice.txt, respons server terhadap Alice, serta penolakan login Eiri.

### Penyelesaian dan Hasil Analisis

1. Konfigurasi FTP menggunakan `vsftpd` pada Chisa (`192.217.2.10`). Akun `alice`, `mika`, dan `eiri` memakai password lab `ndaru`. Perbedaan hak akses diatur pada server, bukan dengan mengganti password.

2. Mulai layanan dari console Chisa dan periksa kebijakannya:

   ```bash
   sh /root/mulai_ftp.sh
   cat /etc/vsftpd.user_list
   cat /etc/vsftpd/users/mika
   ```

   Blacklist harus memuat `eiri`; konfigurasi Mika harus berisi `write_enable=NO`.

3. Aktifkan capture Chisa–Switch2, kemudian gunakan filter:

   ```text
   ftp || ftp-data
   ```

4. Dari Alice, buat dan upload file:

   ```bash
   printf '%s\n' 'Sinyal dari Alice untuk Chisa.' > /root/signal_alice.txt
   curl -v --disable-epsv -u 'alice:ndaru' -T /root/signal_alice.txt ftp://192.217.2.10/signal_alice.txt
   ```

5. Dari Eiri, coba login menggunakan akun yang diblokir:

   ```bash
   curl -v -u 'eiri:ndaru' ftp://192.217.2.10/
   ```

6. Pada Chisa, periksa keberadaan file hasil upload:

   ```bash
   ls -l /var/wired/data/signal_alice.txt
   ```

   Tunjukkan blacklist bersama respons `530`, karena kode `530` juga dapat disebabkan oleh password yang salah.

### Hasil

**Hasil yang diharapkan dari pengujian:**

- **Shared folder:** `/var/wired/data`.
- **Alice:** upload `signal_alice.txt` berhasil, respons `226`, file ada di Chisa.
- **Mika:** `write_enable=NO`; pembuktian pembatasan dilakukan pada Soal 9.
- **Eiri:** login ditolak dengan `530` dan nama akun berada di blacklist.

### Validasi

Upload Alice harus mendapat respons `226` dan menghasilkan file di Chisa. Eiri harus ditolak dengan `530` serta tercantum pada blacklist.

## Soal 8

**Deskripsi Soal:** Knights mengunggah dokumen laporan ke FTP Chisa menggunakan akun alice, kemudian menganalisis perintah upload, respons sukses, dan port data PASV.

**File upload:** `knights_report.txt`

**File capture:** hasil capture praktikum, disarankan `soal08.pcapng`.

**Lokasi capture:** kabel Knights–Switch3.

**Analisis:** perintah STOR, respons 226, ukuran data, dan port data yang dinegosiasikan melalui respons 227.

### Penyelesaian dan Hasil Analisis

1. Aktifkan capture Knights–Switch3. Dari console Knights jalankan:

   ```bash
   wc -c /root/knights_report.txt
   curl -v --disable-epsv -u 'alice:ndaru' -T /root/knights_report.txt ftp://192.217.2.10/knights_report.txt
   ```

2. Gunakan filter `ftp || ftp-data`. Pada capture ditemukan:

   | Nomor paket | Informasi |
   |---|---|
   | 1939 | `STOR knights_report.txt` |
   | 1941 | `FTP Data: 1111 bytes` |
   | 1946 | `226 Transfer complete` |

3. Cari negosiasi passive mode menggunakan filter:

   ```text
   ftp.response.code == 227
   ```

4. Pilih paket **1933**, yaitu respons PASV yang mendahului upload pada paket 1939. Responsnya:

   ```text
   227 Entering Passive Mode (192,217,2,10,118,184)
   ```

   Empat angka pertama menyatakan IP server `192.217.2.10`; dua angka terakhir digunakan untuk menghitung port data.

5. Port data dihitung menggunakan:

   ```text
   port = p1 × 256 + p2
   port = 118 × 256 + 184
   port = 30392
   ```

   Hasil ini cocok dengan port server `30392` pada koneksi TCP data pada capture.

6. Untuk melengkapi verifikasi file tujuan, jalankan di Chisa:

   ```bash
   wc -c /var/wired/data/knights_report.txt
   ```

### Hasil

- **Client:** Knights, `192.217.3.10`.
- **Server:** Chisa, `192.217.2.10`.
- **Akun FTP:** `alice`.
- **Perintah upload:** `STOR knights_report.txt`.
- **Ukuran data pada capture:** `1111` byte.
- **Respons sukses:** `226 Transfer complete`.
- **Respons PASV:** `(192,217,2,10,118,184)`.
- **Port data PASV:** `30392`.

Nomor paket dan port ini berlaku untuk sesi yang dianalisis; port dapat berubah saat upload diulang.

### Validasi

Perintah `STOR`, data 1111 byte, respons `226`, dan respons PASV paket 1933 merujuk sesi yang sama. Periksa file tujuan dengan `wc -c /var/wired/data/knights_report.txt`.

## Soal 9

**Deskripsi Soal:** Mika mengunduh dokumen Protokol Tujuh dari FTP Chisa, kemudian membuktikan akses read-only dengan percobaan upload yang ditolak.

**File download:** `protocol7_manifesto.txt`

**File capture:** hasil capture praktikum, disarankan `soal09.pcapng`.

**Lokasi capture:** kabel Mika–Switch1.

**Analisis:** download menggunakan RETR, ukuran file hasil download, serta respons 550 Permission denied saat upload.

### Penyelesaian dan Hasil Analisis

1. Dokumen asli ditempatkan pada `/var/wired/data/protocol7_manifesto.txt` di Chisa dan harus dapat dibaca akun Mika. File bahan soal yang tersedia berukuran `1738` byte.

2. Aktifkan capture Mika–Switch1 dengan filter `ftp || ftp-data`. Dari Mika jalankan:

   ```bash
   curl -v --disable-epsv -u 'mika:ndaru' -o /root/protocol7_manifesto.txt ftp://192.217.2.10/protocol7_manifesto.txt
   wc -c /root/protocol7_manifesto.txt
   ```

3. Untuk menguji pembatasan, buat file baru dan coba upload dari Mika:

   ```bash
   printf '%s\n' 'Percobaan upload Mika.' > /root/uji_upload_mika.txt
   curl -v --disable-epsv -u 'mika:ndaru' -T /root/uji_upload_mika.txt ftp://192.217.2.10/uji_upload_mika.txt
   ```

4. Periksa respons server saat upload. Hasil yang diminta soal adalah `550 Permission denied`. Jika server membalas `553` atau upload berhasil, bukti belum memenuhi hasil yang diminta.

5. Konfigurasi pembatasan diperiksa pada Chisa:

   ```bash
   cat /etc/vsftpd/users/mika
   ```

   File konfigurasi harus menunjukkan `write_enable=NO`.

### Hasil

**Hasil yang diharapkan dari pengujian:**

- **Akun:** `mika`.
- **Download:** berhasil; file hasil download berukuran `1738` byte.
- **Upload:** ditolak dengan `550 Permission denied`.
- **Hak akses:** hanya membaca.

### Validasi

Akun Mika harus berhasil download dan ditolak saat upload dengan respons `550 Permission denied`.

## Soal 10

**Deskripsi Soal:** Knights mengirim 77 paket ping ke Chisa dengan payload 128 byte dan interval 0,3 detik, kemudian menganalisis ICMP, packet loss, dan RTT.

**File capture:** hasil capture praktikum, disarankan `soal10.pcapng`.

**Lokasi capture:** kabel Knights–Switch3.

**Analisis:** ICMP Type/Code untuk Echo Request dan Echo Reply, jumlah paket, packet loss, dan RTT min/avg/max.

### Penyelesaian dan Hasil Analisis

1. Aktifkan capture Knights–Switch3, kemudian jalankan dari Knights:

   ```bash
   ping -c 77 -s 128 -i 0.3 192.217.2.10
   ```

2. Gunakan filter Wireshark:

   ```text
   icmp && ip.addr == 192.217.3.10 && ip.addr == 192.217.2.10
   ```

3. Klik paket Echo Request dan Echo Reply, lalu buka bagian **Internet Control Message Protocol**. Nilai protokol yang diperiksa adalah Echo Request Type `8`, Code `0`, serta Echo Reply Type `0`, Code `0`.

4. Pada output terminal Knights, statistik ping menunjukkan 77 paket dikirim, 77 diterima, dan packet loss 0%. Nilai RTT terlihat pada baris ringkasan.

5. Payload `-s 128` adalah 128 byte. Tampilan balasan `136 bytes` mencakup header ICMP 8 byte; nilai tersebut bukan ukuran total frame Ethernet.

### Hasil

- **Source:** Knights, `192.217.3.10`.
- **Destination:** Chisa, `192.217.2.10`.
- **Paket dikirim:** `77`.
- **Paket diterima:** `77`.
- **Packet loss:** `0%`.
- **RTT minimum:** `0.366 ms`.
- **RTT rata-rata:** `0.683 ms`.
- **RTT maksimum:** `1.507 ms`.
- **RTT mdev:** `0.210 ms`.
- **Nilai ICMP:** request `8/0`, reply `0/0`.

### Validasi

Sesi yang dianalisis menghasilkan 77 paket diterima dari 77 paket dikirim. Cocokkan Type dan Code ICMP dengan request `8/0` serta reply `0/0`.

## Soal 11

**Deskripsi Soal:** Layanan Telnet di Chisa digunakan untuk membuktikan bahwa kredensial akun phantom_user dengan password wired_ghost dapat terbaca pada komunikasi tanpa enkripsi.

**File capture:** hasil capture praktikum, disarankan `soal11.pcapng`.

**Lokasi capture:** kabel Eiri–Switch3.

**Analisis:** login Telnet, kredensial plaintext pada TCP stream, dan pengiriman ketikan dalam mode interaktif.

### Penyelesaian dan Hasil Analisis

1. Mulai layanan Telnet dari Chisa:

   ```bash
   sh /root/mulai_telnet.sh
   ```

2. Aktifkan capture Eiri–Switch3. Gunakan filter:

   ```text
   tcp.port == 23
   ```

3. Dari console Eiri, hubungkan ke Chisa:

   ```bash
   /bin/busybox-extras telnet 192.217.2.10
   ```

4. Saat diminta, masukkan username `phantom_user` dan password `wired_ghost`. Setelah login, jalankan pada sesi Telnet:

   ```bash
   id
   exit
   ```

5. Pada Wireshark pilih paket sesi Telnet, lalu **Follow > TCP Stream**. Cari karakter username dan password yang dikirim dari client. Tidak munculnya password pada layar terminal saat diketik bukan berarti data jaringan terenkripsi.

6. Telnet bersifat interaktif sehingga ketikan dapat dikirim sedikit demi sedikit. Karakter sering muncul pada paket terpisah, tetapi buffering dan segmentasi TCP dapat membuat lebih dari satu karakter berada dalam sebuah paket.

### Hasil

- **Client:** Eiri.
- **Server:** Chisa, `192.217.2.10:23`.
- **Username sesuai soal:** `phantom_user`.
- **Password sesuai soal:** `wired_ghost`.
- **Sifat protokol:** tidak mengenkripsi sesi Telnet.

### Validasi

Login menggunakan `phantom_user / wired_ghost`, periksa hasil `id`, lalu baca kredensial pada Follow TCP Stream.

## Soal 12

**Deskripsi Soal:** Alice memeriksa port SSH 22 dan HTTP 80 pada Knights dalam keadaan terbuka, serta port 7777 dalam keadaan tertutup menggunakan Netcat.

**File capture:** hasil capture praktikum, disarankan `soal12.pcapng`.

**Lokasi capture:** kabel Alice–Switch1.

**Analisis:** hasil koneksi Netcat serta perbedaan respons SYN-ACK dan RST-ACK dari server.

### Penyelesaian dan Hasil Analisis

1. Pada Knights, periksa listener terlebih dahulu:

   ```bash
   netstat -tlnp
   ```

   Jika 22 dan 80 sudah LISTEN, lanjutkan pengujian. Jika belum, mulai layanan dengan `sh /root/mulai_knights.sh` dan periksa kembali.

2. Pada percobaan sebelumnya muncul `httpd: bind: Address in use`. Pesan ini menunjukkan alamat/port yang hendak dipakai HTTP server sedang digunakan. Periksa proses pada port 80, bukan menyalakan server berulang-ulang. Pesan tersebut sendiri belum membuktikan hasil scan dari Alice.

3. Aktifkan capture Alice–Switch1 dan gunakan filter:

   ```text
   ip.addr == 192.217.3.10 && (tcp.port == 22 || tcp.port == 80 || tcp.port == 7777)
   ```

4. Dari console Alice jalankan:

   ```bash
   nc -zv -w 3 192.217.3.10 22
   nc -zv -w 3 192.217.3.10 80
   nc -zv -w 3 192.217.3.10 7777
   ```

5. Bandingkan paket respons **dari Knights**. Port terbuka membalas SYN-ACK. Port tertutup yang diminta pada soal membalas RST-ACK. Jangan memakai RST penutup koneksi dari Alice sebagai bukti port server tertutup.

### Hasil

**Hasil yang diharapkan dari pengujian:**

| Port | Hasil Netcat | Respons dari Knights |
|---|---|---|
| 22 | succeeded | SYN, ACK |
| 80 | succeeded | SYN, ACK |
| 7777 | Connection refused | RST, ACK |

### Validasi

Hasil Netcat harus sesuai dengan respons dari Knights: SYN-ACK pada port 22/80 dan RST-ACK pada port 7777.

## Soal 13

**Deskripsi Soal:** Administrasi jarak jauh dari Mika ke Knights menggunakan OpenSSH dan public key authentication untuk user mika_admin dengan PasswordAuthentication no.

**File capture:** hasil capture praktikum, disarankan `soal13.pcapng`.

**Lokasi capture:** kabel Mika–Switch1.

**Analisis:** konfigurasi autentikasi, identitas host server, login publickey, Protocol Version Exchange, Key Exchange, dan enkripsi sesi SSH.

### Penyelesaian dan Hasil Analisis

1. Setup menggunakan pasangan kunci `/root/.ssh/demo_modul1_ed25519` di Mika. Public key dipasang pada `/home/mika_admin/.ssh/authorized_keys` di Knights. Private key tetap berada di Mika.

2. Dari Knights periksa konfigurasi efektif dan fingerprint host server:

   ```bash
   /usr/sbin/sshd -T | grep -E 'passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication'
   ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
   ```

   Nilai yang diharapkan: `passwordauthentication no`, `kbdinteractiveauthentication no`, dan `pubkeyauthentication yes`.

3. Aktifkan capture Mika–Switch1 dan gunakan filter `tcp.port == 22`. Dari Mika jalankan:

   ```bash
   ssh -v -i /root/.ssh/demo_modul1_ed25519 -o IdentitiesOnly=yes -o PreferredAuthentications=publickey mika_admin@192.217.3.10
   ```

4. Jika diminta konfirmasi host key, cocokkan fingerprint dengan hasil langsung dari Knights sebelum menjawab `yes`. Jika muncul **REMOTE HOST IDENTIFICATION HAS CHANGED**, periksa bahwa fingerprint server pada pesan tersebut cocok dengan Knights. Hanya jika cocok, hapus catatan lama di Mika:

   ```bash
   ssh-keygen -R 192.217.3.10
   ```

   Ulangi command SSH langkah sebelumnya dan cocokkan fingerprint kembali. Jika tidak cocok, hentikan koneksi dan periksa server tujuan.

5. Setelah login berhasil, jalankan di dalam sesi SSH:

   ```bash
   id
   hostname
   exit
   ```

   Hasil harus menunjukkan user `mika_admin` dan hostname Knights. Log `ssh -v` harus menunjukkan autentikasi menggunakan `publickey`.

6. Di Wireshark, identifikasi **Protocol Version Exchange** dan **Key Exchange**. Setelah pembentukan kunci sesi, isi komunikasi terenkripsi sehingga kredensial tidak terlihat sebagai teks terbuka seperti Telnet.

### Hasil

- **Client:** Mika, `192.217.1.20`.
- **Server:** Knights, `192.217.3.10:22`.
- **User tujuan:** `mika_admin`.
- **Metode yang diminta:** public key authentication.
- **Konfigurasi yang harus terbukti:** `PasswordAuthentication no`.

### Validasi

Setelah identitas server diverifikasi, login harus menggunakan publickey tanpa password akun. Periksa user `mika_admin`, hostname Knights, dan konfigurasi `PasswordAuthentication no`.

# Kesimpulan

LAIN berperan sebagai router untuk tiga subnet dengan forwarding dan NAT agar client dapat saling berkomunikasi serta mengakses internet. FTP menyediakan pengaturan hak akses per akun. Pada sesi upload Knights yang dianalisis, perintah `STOR` berhasil dengan respons `226`, data berukuran `1111` byte, dan port PASV `30392`. Pengujian ping Knights ke Chisa menghasilkan 77 paket diterima tanpa packet loss, dengan RTT rata-rata `0.683 ms`.

Analisis Telnet dan SSH membandingkan komunikasi plaintext dengan komunikasi terenkripsi. Pengujian Netcat membedakan port terbuka dan tertutup melalui respons TCP server. Keberhasilan tiap konfigurasi diperiksa menggunakan langkah validasi pada masing-masing soal.

## Soal 14
**Deskripsi Soal:** Eiri gagal masuk lewat FTP, sehingga coba serangan brute-force ke form login web punya Alice.

**File capture:** `wired_bruteforce.pcapng`

**Analisis:** alamat IP penyerang, target IP beserta port yang diserang, password user `lain_admin`, web server software dan versi yang dilaporkan pada response header.

**Validasi:**
```bash
nc 10.4.89.246 3401
```

### Penyelesaian dan Hasil Analisis
1. Filter dengan `http.request.method == "POST"` di kolom filter untuk menyaring request POST yang digunakan pada proses login.
   
  <img width="846" height="596" alt="Screenshot 2026-09-20 at 13 31 14" src="https://github.com/user-attachments/assets/498783c4-a99a-48a3-bd28-1a86d55b7987" />


2. Terlihat banyak request `POST /login.php` yang dilakukan berulang-ulang ke alamat IP yang sama. Dari informasi paket diperoleh:
   - **Source (IP penyerang):** `172.26.7.50`
   - **Destination (IP target):** `172.26.7.100`
   - **Port tujuan:** `8080`

3. Selanjutnya dilakukan pencarian response login menggunakan filter `http.response.code == 200` untuk menemukan response yang menunjukkan proses login berhasil.

   <img width="1053" height="756" alt="Screenshot 2026-09-20 at 13 49 24" src="https://github.com/user-attachments/assets/a9907f7f-e71b-4538-891a-8ae5d9c5628f" />



4. Pada paket response yang berhasil, dilakukan **Follow > HTTP Stream** untuk melihat isi komunikasi HTTP secara lengkap.

<img width="819" height="578" alt="Screenshot 2026-09-20 at 14 23 15" src="https://github.com/user-attachments/assets/b9f427be-9881-4bab-b548-93d921c8c409" />



5. Dari hasil HTTP Stream ditemukan password untuk user `lain_admin`, yaitu:
   - **Username:** `lain_admin`
   - **Password:** `wired_pr0tocol_7`

6. Pada response header juga ditemukan informasi web server yang digunakan:
   - **Web Server:** `Apache/2.4.62`

### Hasil
- **IP Penyerang:** `172.26.7.50`
- **IP Target:** `172.26.7.100`
- **Port Target:** `8080`
- **Password `lain_admin`:** `wired_pr0tocol_7`
- **Web Server:** `Apache/2.4.62`

### Validasi
Dilakukan validasi menggunakan perintah:

```bash
nc 10.4.89.246 3401
```

Hasil validasi berhasil setelah memasukkan jawaban yang sesuai.

<img width="716" height="474" alt="Screenshot 2026-09-20 at 20 19 46" src="https://github.com/user-attachments/assets/abcf0228-2621-49fc-9310-cff50a0a305e" />



## Soal 15
**Deskripsi Soal:** Analisis komunikasi USB HID untuk menemukan informasi perangkat USB dan pesan rahasia yang dikirimkan.

**File capture:** `wired_usb_hid.pcap`

**Analisis:** Vendor ID, Product ID, USB device address, dan secret message.

**Validasi:**
```bash
nc 10.4.89.246 3402
```

### Penyelesaian dan Hasil Analisis
1. Buka file capture `wired_usb_hid.pcap` menggunakan Wireshark kemudian gunakan filter:
   
   `usb`

   <img width="773" height="556" alt="Screenshot 2026-09-20 at 20 26 16" src="https://github.com/user-attachments/assets/cfa06742-60ae-4175-af4e-ac29d50784a2" />


2. Dari packet details ditemukan informasi perangkat USB berupa:
   - **Vendor ID:** `0x046d`
   - **Product ID:** `0xc31c`
   - **USB Device Address:** `7`

3. Selanjutnya dilakukan analisis pada data HID untuk mengetahui karakter yang dikirimkan oleh perangkat.

   <img width="627" height="444" alt="Screenshot 2026-09-20 at 20 30 54" src="https://github.com/user-attachments/assets/73149e6f-e77a-4364-9fa3-1a6164df573f" />


4. Data HID kemudian diterjemahkan berdasarkan USB HID Usage Tables. Perlu memperhatikan nilai modifier karena `0x02` menunjukkan penggunaan **Left Shift**.
<img width="627" height="444" alt="Screenshot 2026-09-20 at 20 30 54" src="https://github.com/user-attachments/assets/7406bd47-7ea9-4dc7-8461-bc77ccb446fa" />


5. Setelah seluruh data HID diterjemahkan, diperoleh pesan rahasia:
   
   `Wired_Protocol_7_is_alive_2026`

### Hasil
- **Vendor ID:** `0x046d`
- **Product ID:** `0xc31c`
- **USB Device Address:** `7`
- **Secret Message:** `Wired_Protocol_7_is_alive_2026`

### Validasi
Dilakukan validasi menggunakan:

```bash
nc 10.4.89.246 3402
```

<img width="561" height="380" alt="Screenshot 2026-09-20 at 20 37 17" src="https://github.com/user-attachments/assets/3db5f76f-097c-4136-b0bc-2288ba47666a" />



## Soal 16
**Deskripsi Soal:** Analisis traffic FTP untuk menemukan server FTP, software yang digunakan, kredensial login, dan ukuran file `knights_payload.exe`.

**File capture:** `wired_ftp_theft.pcap`

**Analisis:** IP server FTP, FTP software banner, username dan password attacker, serta ukuran file `knights_payload.exe`.

**Validasi:**
```bash
nc 10.4.89.246 3403
```

### Penyelesaian dan Hasil Analisis
1. Gunakan filter `ftp` pada Wireshark untuk menampilkan komunikasi FTP.

   <img width="620" height="454" alt="Screenshot 2026-09-20 at 21 04 59" src="https://github.com/user-attachments/assets/f45054f9-e6a6-40f7-bf68-b1dae956fc58" />


2. Untuk menemukan proses pengambilan file, gunakan filter:

   `ftp.request.command == "RETR"`

3. Pada packet ditemukan request:

   `RETR knights_payload.exe`

   dengan komunikasi:
   - **Source:** `10.7.3.50`
   - **Destination:** `198.51.100.7`

   Sehingga IP server FTP adalah `198.51.100.7`.

<img width="620" height="454" alt="Screenshot 2026-09-20 at 21 04 59" src="https://github.com/user-attachments/assets/4a9cd2c0-943a-483f-b4fc-9cc2370a2a65" />


4. Selanjutnya dilakukan **Follow > TCP Stream** untuk melihat banner FTP.

   Ditemukan banner:

   `220 Welcome to Wired FTP Server (vsftpd 3.0.5)`

   Sehingga software FTP yang digunakan adalah:
   - **FTP Software:** `vsftpd 3.0.5`


5. Pada komunikasi FTP ditemukan kredensial login:

   - **Username:** `knights_agent`
   - **Password:** `N4v1_s3cur3_2026`

6. Dari command FTP `SIZE` ditemukan:

   `SIZE knights_payload.exe`

   dengan response:

   `213 524288`

   Sehingga ukuran file adalah `524288` byte.

### Hasil
- **FTP Server IP:** `198.51.100.7`
- **FTP Software:** `vsftpd 3.0.5`
- **Username:** `knights_agent`
- **Password:** `N4v1_s3cur3_2026`
- **Credentials:** `knights_agent:N4v1_s3cur3_2026`
- **File Size:** `524288` byte

### Validasi
Dilakukan validasi menggunakan:

```bash
nc 10.4.89.246 3403
```

<img width="577" height="327" alt="Screenshot 2026-09-20 at 21 08 12" src="https://github.com/user-attachments/assets/d6e90391-84f4-4454-a1c7-02e5e2c18451" />



## Soal 17
**Deskripsi Soal:** Analisis komunikasi HTTP untuk menemukan domain host, IP server attacker, nama executable, dan HTTP status code.

**File capture:** `wired_http_c2.pcap`

**Analisis:** Host domain, attacker server IP, executable filename, dan HTTP status code.

**Validasi:**
```bash
nc 10.4.89.246 3404
```

### Penyelesaian dan Hasil Analisis
1. Gunakan filter `http` pada Wireshark untuk menampilkan seluruh komunikasi HTTP.

   <img width="621" height="460" alt="Screenshot 2026-09-20 at 21 15 40" src="https://github.com/user-attachments/assets/7f1dfb7f-33d8-441c-8cf1-dd641e274c8d" />


2. Dari beberapa HTTP request, ditemukan request menuju server:

   `GET /navi_agent.exe`

3. Pada packet tersebut diketahui:
   - **Source:** `10.7.1.50`
   - **Destination:** `203.0.113.42`
   - **Destination Port:** `80`
   - **Host:** `wired-update.net`

 

4. Dari request tersebut ditemukan nama executable yang diminta:

   `navi_agent.exe`

5. Selanjutnya dilihat response dari server. Pada response ditemukan:

   `HTTP/1.1 200 OK`

   sehingga HTTP status code yang diperoleh adalah `200`.
<img width="559" height="592" alt="Screenshot 2026-09-20 at 21 24 13" src="https://github.com/user-attachments/assets/42784224-ae70-4545-b06e-729eeb45b108" />

   

6. Informasi tambahan pada response menunjukkan server menggunakan `nginx/1.24.0`.

### Hasil
- **Host Domain:** `wired-update.net`
- **Attacker Server IP:** `203.0.113.42`
- **Executable Filename:** `navi_agent.exe`
- **HTTP Status Code:** `200`

### Validasi
Dilakukan validasi menggunakan:

```bash
nc 10.4.89.246 3404
```

<img width="562" height="434" alt="Screenshot 2026-09-20 at 21 22 17" src="https://github.com/user-attachments/assets/00852f80-7b1a-4b1d-a85a-72069c495c61" />



## Soal 18
**Deskripsi Soal:** Analisis transfer file menggunakan SMB untuk mengetahui protokol jaringan, IP pengirim dan penerima, folder tujuan malware, serta nama executable.

**File capture:** `wired_smb_transfer.pcapng`

**Analisis:** network protocol, sender IP, receiver IP, malware destination folder, dan executable filename.

**Validasi:**
```bash
nc 10.4.89.246 3405
```

### Penyelesaian dan Hasil Analisis
1. Gunakan filter `smb2` pada Wireshark untuk menampilkan komunikasi SMB2.

 <img width="692" height="506" alt="Screenshot 2026-09-20 at 21 25 02" src="https://github.com/user-attachments/assets/156621ba-d743-4945-a8da-14d9bb47fb14" />


2. Pada packet **Create Request** ditemukan komunikasi:

   - **Source:** `10.7.3.100`
   - **Destination:** `10.7.1.50`
   - **File:** `System32\wired_trojan_payload.exe`



3. Pada packet **Write Request** juga ditemukan file yang sama:

   `System32\wired_trojan_payload.exe`

   Hal ini menunjukkan adanya proses penulisan/transfer file menuju host penerima.

   <img width="1042" height="756" alt="Screenshot 2026-09-20 at 21 27 37" src="https://github.com/user-attachments/assets/db5a51bc-17c7-4bec-b0df-e0e497049a13" />


4. Berdasarkan hasil analisis packet, diperoleh informasi bahwa protokol yang digunakan adalah SMB2.

### Hasil
- **Network Protocol:** `SMB2`
- **Sender IP:** `10.7.3.100`
- **Receiver IP:** `10.7.1.50`
- **Destination Folder:** `System32`
- **Executable Filename:** `wired_trojan_payload.exe`

### Validasi
Dilakukan validasi menggunakan:

```bash
nc 10.4.89.246 3405
```

<img width="820" height="344" alt="Screenshot 2026-09-20 at 21 28 16" src="https://github.com/user-attachments/assets/fa2303dd-468c-43b4-8966-88d0066fe8f3" />



## Soal 19
**Deskripsi Soal:** Analisis komunikasi SMTP untuk menemukan email korban, password yang diklaim bocor, jenis malware, batas waktu pembayaran, dan MailClientID.

**File capture:** `wired_smtp_threat.pcap`

**Analisis:** victim email, claimed leaked password, malware type, deadline, dan MailClientID.

**Validasi:**
```bash
nc 10.4.89.246 3406
```

### Penyelesaian dan Hasil Analisis
1. Gunakan filter `smtp` pada Wireshark untuk menampilkan komunikasi SMTP.

  <img width="833" height="612" alt="Screenshot 2026-09-20 at 21 32 31" src="https://github.com/user-attachments/assets/8921f28d-46f9-4cbe-8249-a70e0ee294f2" />


2. Selanjutnya gunakan filter:

   `smtp.req.command == "DATA"`

   untuk menemukan paket yang berisi isi email.

3. Ditemukan beberapa komunikasi email. Setelah dilakukan pemeriksaan terhadap masing-masing isi email, komunikasi pada packet 84 merupakan email ancaman yang berisi informasi pemerasan.

 <img width="687" height="501" alt="Screenshot 2026-09-20 at 21 33 10" src="https://github.com/user-attachments/assets/d63db1a4-447a-48fd-9369-ade4fce43d28" />


4. Pada packet tersebut dilakukan **Follow > TCP Stream** untuk membaca isi email secara lengkap.

   <img width="449" height="485" alt="Screenshot 2026-09-20 at 21 33 50" src="https://github.com/user-attachments/assets/c0494dd8-e61c-41f3-b6dc-0819714242c5" />


5. Dari isi email ditemukan informasi:
   - **Victim Email:** `victim@protocol7.co.jp`
   - **Claimed Leaked Password:** `pr0tocol_7_user`
   - **Malware Type:** `ransomware`
   - **Deadline:** `3 days`
   - **MailClientID:** `7719980706`

### Hasil
- **Victim Email:** `victim@protocol7.co.jp`
- **Claimed Leaked Password:** `pr0tocol_7_user`
- **Malware Type:** `ransomware`
- **Deadline:** `3 days`
- **MailClientID:** `7719980706`

### Validasi
Dilakukan validasi menggunakan:

```bash
nc 10.4.89.246 3406
```

<img width="497" height="448" alt="Screenshot 2026-09-20 at 21 34 22" src="https://github.com/user-attachments/assets/05915fa0-0068-4908-8103-cfdf9b9712e2" />



## Soal 20
**Deskripsi Soal:** Analisis traffic TLS menggunakan file key log untuk mengetahui informasi TLS dan membuka isi HTTP yang terenkripsi.

**File capture:** `wired_tls_decrypt.pcapng`

**File key log:** `keyslogfile.txt`

**Analisis:** TLS version, SNI, attacker HTTPS server IP, User-Agent, HTTP method, dan HTTP path.

**Validasi:**
```bash
nc 10.4.89.246 3407
```

### Penyelesaian dan Hasil Analisis
1. Buka file `wired_tls_decrypt.pcapng` menggunakan Wireshark kemudian gunakan filter:

   `tls`

   <img width="1048" height="761" alt="Screenshot 2026-09-20 at 21 38 04" src="https://github.com/user-attachments/assets/2b751abc-9480-4a4c-bf8f-dd816abe4c37" />


2. Pada packet Client Hello ditemukan informasi SNI:

   `example.com`

   <img width="1048" height="759" alt="Screenshot 2026-09-20 at 21 41 09" src="https://github.com/user-attachments/assets/0f4a5445-5343-424a-885f-6b84cfb5f519" />


3. Pada bagian detail TLS ditemukan versi:

   `TLS 1.2 (0x0303)`

   sehingga TLS version yang digunakan adalah `TLS 1.2`.

4. Dari komunikasi TLS diketahui koneksi HTTPS menuju:
   - **Source:** `10.9.0.2`
   - **Destination:** `93.184.216.34`
   - **Destination Port:** `443`

   Sehingga IP HTTPS server adalah `93.184.216.34`.

5. Untuk membuka isi komunikasi HTTPS, masuk ke:

   **Wireshark → Preferences → Protocols → TLS**

   kemudian pada bagian **(Pre)-Master-Secret log filename**, pilih file:

   `keyslogfile.txt`

   Setelah itu klik **Apply/OK**.

 <img width="680" height="475" alt="Screenshot 2026-09-20 at 21 42 09" src="https://github.com/user-attachments/assets/cc12fa91-b853-4df7-9839-1d66aad4e8b8" />


6. Setelah key log berhasil digunakan, filter `http` dapat digunakan untuk melihat HTTP yang sebelumnya terenkripsi.


7. Pada packet HTTP ditemukan:

   `HEAD / HTTP/1.1`

   dengan informasi:

   - **Host:** `example.com`
   - **User-Agent:** `curl/7.62.0`
   - **Accept:** `*/*`

   

8. Dari request tersebut diketahui HTTP method dan path yang tersembunyi di dalam komunikasi TLS:
   - **HTTP Method:** `HEAD`
   - **HTTP Path:** `/`

### Hasil
- **TLS Version:** `TLS 1.2`
- **SNI:** `example.com`
- **HTTPS Server IP:** `93.184.216.34`
- **User-Agent:** `curl/7.62.0`
- **Hidden HTTP Method:** `HEAD`
- **Hidden HTTP Path:** `/`

### Validasi
Dilakukan validasi menggunakan:

```bash
nc 10.4.89.246 3407
```

<img width="566" height="250" alt="Screenshot 2026-09-20 at 21 42 59" src="https://github.com/user-attachments/assets/8171ee54-6eec-4b2a-b745-c429ffad9f48" />



# Kesimpulan

Berdasarkan analisis terhadap file capture pada soal 14 sampai 20, informasi yang diminta dapat ditemukan melalui analisis packet menggunakan Wireshark dan beberapa teknik seperti filtering protocol, melihat packet details, melakukan **Follow TCP Stream**, serta melakukan dekripsi TLS menggunakan key log. Setiap hasil analisis kemudian divalidasi menggunakan layanan `nc` pada masing-masing port validasi yang telah disediakan.

Hasil akhir yang diperoleh adalah:

| Soal | Hasil Utama |
|---|---|
| 14 | `172.26.7.50` → `172.26.7.100:8080`, password `wired_pr0tocol_7`, Apache `2.4.62` |
| 15 | Vendor `0x046d`, Product `0xc31c`, Device Address `7`, secret `Wired_Protocol_7_is_alive_2026` |
| 16 | FTP `198.51.100.7`, `vsftpd 3.0.5`, `knights_agent:N4v1_s3cur3_2026`, `524288` byte |
| 17 | `wired-update.net`, `203.0.113.42`, `navi_agent.exe`, status `200` |
| 18 | SMB2, `10.7.3.100` → `10.7.1.50`, `System32`, `wired_trojan_payload.exe` |
| 19 | `victim@protocol7.co.jp`, `pr0tocol_7_user`, ransomware, `3 days`, MailClientID `7719980706` |
| 20 | TLS 1.2, SNI `example.com`, `93.184.216.34`, `curl/7.62.0`, `HEAD /` |

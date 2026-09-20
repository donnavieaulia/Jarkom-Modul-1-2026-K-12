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

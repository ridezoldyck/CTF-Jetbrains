# Investigasi Insiden JetBrains TeamCity (Blue Team CTF)

> **Proyek Portofolio | Blue Team / Respons Insiden / Analisis PCAP**  
> Lab: CyberDefenders — **JetBrains**  
> Peralatan: Wireshark, analisis aliran HTTP/TCP, MITRE ATT&CK

## Gambaran Umum

Proyek ini mendokumentasikan investigasi Blue Team terhadap simulasi kompromi server JetBrains TeamCity. Investigasi menggunakan bukti tangkapan lalu lintas jaringan untuk merekonstruksi aktivitas penyerang, mengidentifikasi layanan yang rentan, melacak penggunaan web shell, serta mendokumentasikan upaya mempertahankan akses dan memperoleh akses ke sistem host.

Proyek ini merupakan **latihan di lingkungan lab terkontrol**, bukan investigasi terhadap sistem produksi. Informasi autentikasi sensitif dan token Bearer sengaja tidak disertakan dalam portofolio publik ini.

## Ringkasan Eksekutif

Berkas PCAP menunjukkan aktivitas HTTP dari sumber eksternal yang mengakses server TeamCity. Server mengungkapkan versinya sebagai **`2023.11.3 (build 147512)`**. Pola permintaan yang diamati konsisten dengan eksploitasi **CVE-2024-27198**, yaitu kerentanan *authentication bypass* pada TeamCity.

Penyerang mengunggah arsip plugin ZIP yang berisi web shell JSP, kemudian mengirimkan permintaan POST ke berkas JSP tersebut untuk menjalankan perintah. Aktivitas tambahan mencakup penulisan kredensial palsu ke berkas teks dan upaya menggunakan Docker untuk mengakses sistem berkas host dari dalam kontainer.

## Lingkungan dan Bukti

| Komponen | Temuan |
|---|---|
| Alamat IP sumber penyerang | `23.158.56.196` |
| Alamat IP server TeamCity dalam PCAP | `172.31.25.119` |
| Layanan HTTP | `3.71.79.4:8111`, terlihat pada header Host permintaan |
| Versi TeamCity | `2023.11.3 (build 147512)` |
| Kerentanan yang dicurigai | `CVE-2024-27198` |
| Arsip yang diunggah | `NSt8bHTg.zip` |
| Lokasi web shell | `/plugins/NSt8bHTg/NSt8bHTg.jsp` |
| Waktu perintah web shell pertama | `2024-06-30 08:03` (jawaban yang diterima pada tantangan; zona waktu perlu diverifikasi jika dianalisis ulang) |
| Berkas yang ditulis penyerang | `/tmp/Creds.txt` |
| Subteknik MITRE ATT&CK | `T1565.001 — Stored Data Manipulation` (Manipulasi Data Tersimpan) |

> **Catatan:** Tangkapan jaringan memuat beberapa representasi alamat dan header karena konfigurasi jaringan serta mekanisme proksi di lab. Alamat di atas merupakan artefak yang diamati dalam PCAP dan bukan bukti atribusi terhadap pelaku di dunia nyata.

## Kronologi Investigasi

### 1. Identifikasi layanan dan versi

Permintaan ke `/hax?jsp=/app/rest/server;.jsp` menghasilkan respons XML yang memuat informasi versi server:

`2023.11.3 (build 147512)`

Informasi ini digunakan sebagai dasar untuk mengidentifikasi kerentanan yang relevan.

### 2. Korelasi kerentanan

Manipulasi URI dan versi TeamCity dikorelasikan dengan `CVE-2024-27198`. Dalam insiden nyata, versi yang terdampak dan status pembaruan harus diverifikasi menggunakan advisori resmi vendor.

### 3. Aktivitas akun

Permintaan ke `/hax?jsp=/app/rest/users;.jsp` memperlihatkan data akun dalam percakapan HTTP. Informasi kata sandi dan token Bearer tidak ditampilkan dalam bukti publik demi menjaga keamanan data autentikasi.

### 4. Pengiriman web shell

Permintaan HTTP ke `/admin/pluginUpload.html` memuat arsip ZIP bernama `NSt8bHTg.zip`. Konten arsip yang terlihat dalam aliran jaringan mencakup berkas `NSt8bHTg.jsp`.

### 5. Eksekusi perintah

Penyerang mengirimkan permintaan HTTP POST ke:

`/plugins/NSt8bHTg/NSt8bHTg.jsp`

Salah satu permintaan yang diamati memuat parameter `cmd=whoami`. Waktu eksekusi pertama yang tercatat sebagai jawaban tantangan adalah `2024-06-30 08:03`.

### 6. Manipulasi berkas kredensial

Perintah melalui web shell menuliskan teks kredensial yang menyesatkan ke `/tmp/Creds.txt`. Aktivitas ini dipetakan ke MITRE ATT&CK `T1565.001 — Stored Data Manipulation` atau manipulasi data tersimpan.

### 7. Upaya mengakses host melalui kontainer

Sebuah perintah Docker berupaya memasang sistem berkas host dan menjalankan `chroot`. Perilaku ini mengindikasikan upaya mengakses sistem host dari dalam kontainer.

Perintah tersebut terlihat pada isi formulir HTTP. Namun, keberhasilan akses terhadap host tidak boleh diasumsikan tanpa bukti tambahan dari telemetri sistem host.

## Indikator Kompromi (IOC)

| Jenis | Nilai | Konteks |
|---|---|---|
| IPv4 | `23.158.56.196` | Sumber permintaan HTTP penyerang yang diamati |
| IPv4 | `172.31.25.119` | Alamat tujuan yang terlihat pada paket terkait |
| Pola URI | `/hax?jsp=/app/rest/server;.jsp` | Manipulasi endpoint informasi server |
| Pola URI | `/hax?jsp=/app/rest/users;.jsp` | Manipulasi endpoint API pengguna |
| Jalur URI | `/admin/pluginUpload.html` | Endpoint pengunggahan plugin atau arsip |
| Nama berkas | `NSt8bHTg.zip` | Arsip yang diunggah |
| Web shell | `/plugins/NSt8bHTg/NSt8bHTg.jsp` | Endpoint eksekusi perintah melalui JSP |
| Jalur berkas | `/tmp/Creds.txt` | Berkas yang dimodifikasi melalui perintah shell |
| CVE | `CVE-2024-27198` | Kerentanan yang dikorelasikan dengan perilaku yang diamati |

**Catatan penanganan:** Indikator tersebut berasal dari lab. Jangan memblokir alamat atau menyelidiki aset produksi hanya berdasarkan tangkapan jaringan latihan ini tanpa validasi tambahan.

## Pemetaan MITRE ATT&CK

| Teknik | Nama | Bukti atau alasan |
|---|---|---|
| `T1190` | Exploit Public-Facing Application | Manipulasi endpoint TeamCity yang berkaitan dengan dugaan eksploitasi CVE |
| `T1505.003` | Server Software Component: Web Shell | Web shell JSP diunggah dan diakses melalui HTTP POST |
| `T1059.004` | Unix Shell | Permintaan web shell membawa perintah shell untuk dieksekusi |
| `T1565.001` | Stored Data Manipulation | Perintah menulis data kredensial yang menyesatkan ke `Creds.txt` |
| `T1610` | Deploy Container | Perintah Docker terlihat dalam aktivitas percobaan akses host |

Pemetaan MITRE ATT&CK di atas merupakan interpretasi analitis berdasarkan PCAP yang terlihat dan konteks tantangan. Beberapa pemetaan perlu ditinjau kembali apabila log host atau keseluruhan PCAP memberikan konteks tambahan.

## Rekomendasi Deteksi dan Respons

- Perbarui TeamCity ke versi yang didukung vendor dan telah memperbaiki CVE-2024-27198. Verifikasi status perbaikan melalui advisori resmi JetBrains.
- Tinjau log akses TeamCity untuk menemukan pola `/hax?jsp=`, pengunggahan plugin yang tidak dikenal, dan akses mencurigakan ke jalur `/plugins/`.
- Cari nama arsip dan jalur JSP yang tercantum dalam IOC pada log web server dan telemetri endpoint.
- Periksa direktori plugin TeamCity untuk menemukan berkas JSP yang tidak sah atau tidak dikenal. Simpan stempel waktu dan hash berkas sebelum melakukan penghapusan.
- Tinjau pembuatan proses, riwayat shell, log runtime kontainer, serta akses ke Docker socket untuk mencari aktivitas yang sesuai dengan pola perintah yang diamati.
- Verifikasi apakah `/tmp/Creds.txt` atau berkas terkait kredensial lainnya dibuat atau dimodifikasi. Ganti kredensial apabila terdapat bukti paparan.
- Cabut token yang terekspos, atur ulang kredensial terdampak, dan tinjau aktivitas pembuatan atau perubahan akun.
- Simpan PCAP asli dan catat hash kriptografisnya sebelum melakukan analisis lanjutan.

## Langkah Analisis Ulang di Wireshark

Gunakan filter berikut untuk menelusuri aktivitas utama dalam PCAP.

**1. Menampilkan lalu lintas HTTP dari sumber yang dicurigai**

```wireshark
ip.src == 23.158.56.196 && http
```

**2. Menemukan permintaan manipulasi endpoint informasi server**

```wireshark
http.request.uri contains "/hax?jsp=/app/rest/server"
```

**3. Menemukan permintaan pengunggahan plugin**

```wireshark
http.request.uri contains "/admin/pluginUpload.html"
```

**4. Menemukan permintaan POST ke web shell**

```wireshark
ip.src == 23.158.56.196 && http.request.method == POST && http.request.uri contains "NSt8bHTg.jsp"
```

**5. Menelusuri lalu lintas yang memuat referensi `Creds.txt`**

```wireshark
ip.src == 23.158.56.196 && http contains "Creds.txt"
```

Untuk setiap paket yang relevan, pilih **Follow → TCP Stream** guna memeriksa arah permintaan dan respons. Hindari memasukkan token Bearer, kata sandi, cookie, atau rahasia lainnya ke dalam dokumentasi publik.

## Bukti Tangkapan Layar

Direktori `evidence/` berisi beberapa tangkapan layar pilihan dari lingkungan latihan. Bagian yang memuat token sensitif atau respons akun telah disamarkan pada salinan yang disertakan.

PCAP asli tidak disertakan dalam paket portofolio ini.

## Keterbatasan Investigasi

- Temuan didasarkan pada tangkapan layar tantangan dan cuplikan aliran HTTP yang terlihat, bukan analisis independen terbaru terhadap PCAP asli secara menyeluruh.
- Dampak akhir dari perintah Docker terhadap sistem host belum terkonfirmasi hanya melalui bukti jaringan.
- Waktu perintah pertama dicatat sesuai jawaban yang diterima oleh tantangan. Konversi zona waktu perlu didokumentasikan apabila PCAP dibuka dan diverifikasi secara independen.
- Kata sandi, token Bearer, dan informasi autentikasi sensitif lainnya tidak disertakan dalam portofolio publik.

## Deskripsi Repository GitHub

> Investigasi PCAP Blue Team terhadap simulasi kompromi JetBrains TeamCity, mencakup korelasi CVE, analisis web shell, kronologi serangan, indikator kompromi, dan pemetaan MITRE ATT&CK.

## Referensi

- CyberDefenders — Lab JetBrains Blue Team CTF: https://cyberdefenders.org/blueteam-ctf-challenges/jetbrains/
- Advisori keamanan JetBrains: https://www.jetbrains.com/privacy-security/issues-fixed/
- MITRE ATT&CK: https://attack.mitre.org/
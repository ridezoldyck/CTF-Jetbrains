# Laporan Insiden — Lab JetBrains TeamCity

## 1. Informasi Kasus

- **Kasus:** Simulasi penyusupan melalui web shell pada TeamCity
- **Jenis:** Analisis PCAP oleh tim Blue Team dan latihan respons insiden
- **Status:** Selesai di lab CyberDefenders
- **Bukti utama:** `Capture.pcap` (digunakan di lab daring dan tidak disertakan dalam repositori ini)

## 2. Ringkasan Insiden

Bukti lalu lintas jaringan menunjukkan adanya permintaan HTTP mencurigakan yang ditujukan ke layanan TeamCity, pengungkapan versi layanan, pengunggahan berkas ZIP melalui fitur plugin, serta eksekusi perintah melalui web shell berbasis JSP.

Pola aktivitas tersebut konsisten dengan eksploitasi kerentanan *authentication bypass* pada TeamCity, yaitu **CVE-2024-27198**. Penyerang juga berupaya memanipulasi berkas teks yang berisi kredensial dan menjalankan perintah Docker yang bertujuan memperoleh akses ke sistem berkas host.

Analisis ini dilakukan untuk mengidentifikasi aktivitas penyerang, memahami tahapan serangan, mendokumentasikan indikator kompromi, dan memberikan rekomendasi penanganan insiden.

## 3. Temuan Utama

- **Versi TeamCity:** `2023.11.3 (build 147512)`, berdasarkan informasi XML yang ditemukan dalam lalu lintas jaringan.
- **Alamat IP sumber penyerang:** `23.158.56.196`.
- **Endpoint pengunggahan:** `/admin/pluginUpload.html`.
- **Nama berkas arsip yang diunggah:** `NSt8bHTg.zip`.
- **Web shell:** `/plugins/NSt8bHTg/NSt8bHTg.jsp`.
- **Aktivitas eksekusi perintah:** Parameter perintah pada permintaan web shell mencakup `whoami`.
- **Manipulasi berkas:** Ditemukan perintah shell yang menulis data kredensial palsu ke `/tmp/Creds.txt`.
- **Percobaan akses sistem berkas host:** Ditemukan perintah Docker yang melibatkan pemasangan sistem berkas host dan penggunaan `chroot`. Namun, keberhasilan memperoleh akses ke host belum dapat dipastikan hanya berdasarkan tangkapan layar yang tersedia.

## 4. Analisis dan Penilaian Insiden

Kombinasi manipulasi endpoint, pengunggahan plugin ZIP, penggunaan web shell JSP, eksekusi perintah, dan perubahan isi berkas menunjukkan terjadinya simulasi kompromi server.

Web shell merupakan komponen berisiko tinggi karena dapat memungkinkan penyerang menjalankan perintah dari jarak jauh menggunakan hak akses proses layanan yang menjalankannya.

Kerentanan CVE-2024-27198 memungkinkan penyerang melewati mekanisme autentikasi dalam kondisi yang rentan. Dalam kasus ini, aktivitas yang teramati sesuai dengan pola eksploitasi yang memanfaatkan kelemahan tersebut untuk melanjutkan akses ke layanan TeamCity.

Meskipun demikian, setiap kesimpulan harus didasarkan pada bukti yang tersedia. Keberadaan perintah dalam lalu lintas jaringan tidak dengan sendirinya membuktikan bahwa perintah tersebut berhasil dijalankan atau bahwa penyerang berhasil memperoleh kendali penuh atas host.

## 5. Rekomendasi Penanganan Insiden

1. **Isolasi sistem:** Jika kejadian serupa ditemukan di lingkungan produksi, batasi konektivitas sistem yang terdampak untuk mencegah aktivitas lanjutan.
2. **Amankan bukti:** Simpan PCAP, log aplikasi, stempel waktu berkas, serta informasi proses dan kontainer yang relevan untuk kebutuhan investigasi.
3. **Perbarui sistem:** Terapkan pembaruan TeamCity sesuai rekomendasi resmi vendor dan verifikasi bahwa versi yang terpasang tidak lagi rentan.
4. **Periksa artefak mencurigakan:** Identifikasi dan karantina plugin atau web shell yang tidak sah setelah bukti yang diperlukan diamankan.
5. **Rotasi kredensial:** Ganti kredensial yang berpotensi terekspos dan cabut token sesi atau token autentikasi yang mungkin telah disalahgunakan.
6. **Lakukan pencarian ancaman:** Telusuri indikator kompromi pada log proxy, web server, endpoint, aplikasi, dan kontainer.
7. **Verifikasi akses host:** Gunakan telemetri host, log Docker, riwayat proses, dan bukti sistem berkas untuk menentukan apakah percobaan akses melalui Docker dan `chroot` berhasil.
8. **Dokumentasikan hasil:** Catat kronologi, bukti pendukung, tingkat keyakinan setiap temuan, tindakan mitigasi, serta hasil verifikasi setelah perbaikan.

## 6. Perlindungan Informasi Sensitif

Repositori portofolio ini tidak menyertakan kata sandi, token Bearer, cookie sesi, atau header HTTP sensitif dalam bentuk mentah.

Berkas PCAP asli tidak dipublikasikan karena dapat memuat informasi autentikasi dan data lain yang bersifat sensitif. Berkas tersebut hanya boleh dibagikan apabila lisensi dan ketentuan penggunaan lab secara eksplisit mengizinkannya.

## 7. Keterampilan yang Dipraktikkan

Melalui latihan ini, keterampilan yang dipraktikkan meliputi:

- Analisis lalu lintas jaringan menggunakan Wireshark.
- Investigasi permintaan dan respons HTTP.
- Identifikasi versi layanan dan korelasi kerentanan.
- Analisis pengunggahan berkas dan aktivitas web shell.
- Identifikasi aktivitas eksekusi perintah mencurigakan.
- Pemetaan perilaku penyerang ke MITRE ATT&CK.
- Penyusunan indikator kompromi dan laporan insiden.
- Penyusunan rekomendasi mitigasi serta pemulihan.

## 8. Kesimpulan

Latihan ini menunjukkan bagaimana analisis lalu lintas jaringan dapat digunakan untuk merekonstruksi tahapan serangan terhadap layanan TeamCity, mulai dari identifikasi layanan dan eksploitasi kerentanan hingga pengunggahan web shell dan percobaan eksekusi perintah.

Hasil investigasi didokumentasikan sebagai proyek pembelajaran Blue Team untuk menunjukkan kemampuan analisis PCAP, investigasi insiden, identifikasi indikator kompromi, dan penyusunan rekomendasi keamanan.

**Catatan:** Proyek ini merupakan simulasi investigasi dalam lab CyberDefenders, bukan bukti bahwa sistem produksi benar-benar telah disusupi.
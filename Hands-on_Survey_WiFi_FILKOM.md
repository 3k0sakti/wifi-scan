# Hands-on: Pemetaan Kualitas Sinyal Wi-Fi 5 GHz Gedung FILKOM

**Mata Kuliah:** Jaringan Nirkabel
**Bentuk:** Praktikum lapangan berkelompok, **2 pertemuan (3 SKS)**. Minggu ini eksperimen, minggu depan presentasi.
**Kelompok:** 3 kelompok. **Kelompok 1 → Lantai 2**, **Kelompok 2 → Lantai 3**, **Kelompok 3 → Lantai 4**
**Fokus:** Band **5 GHz** (band yang digunakan jaringan FILKOM saat ini)
**Perangkat:** Laptop **Windows** atau **Linux** + ponsel Android
**Status dokumen:** DRAFT v0.3

---

## Ringkasan Tugas

Setiap kelompok menggambar sendiri **denah lantainya**, lalu melakukan **site survey Wi-Fi 5 GHz** untuk menjawab tiga pertanyaan:

1. **Di mana saja spot yang kualitas sinyalnya kurang baik?** (blank spot, sinyal lemah, interferensi, kanal padat)
2. **Mengapa spot tersebut buruk?** (jauh dari AP, terhalang dinding, co-channel interference, beban, tidak ada AP cadangan untuk roaming, sinyal "bocor" dari lantai lain)
3. **Bagaimana peta penempatan AP saat ini, dan bagaimana seharusnya?** (posisi AP, kanal, dan usulan perbaikan)

Produk utama setiap kelompok adalah **Peta Kualitas Sinyal dan Penempatan AP** untuk lantainya, yang dipresentasikan pada pertemuan berikutnya dan digabung menjadi gambaran kondisi seluruh gedung.

### Jadwal dan beban 3 SKS

| Waktu | Bentuk | Kegiatan | Hasil |
|---|---|---|---|
| **Sebelum Pertemuan 1** | Mandiri | Instalasi tools di laptop, uji scan, soal pendahuluan (§3) | Laptop siap |
| **Pertemuan 1** (minggu ini, 150 menit) | Tatap muka, lapangan | Sketsa denah, kalibrasi, survei pasif, survei aktif, roaming (§7.1) | Sketsa denah + data mentah |
| **Selama seminggu** | Terstruktur | Digitalisasi denah, koordinat titik, analisis, laporan, dan slide. Boleh survei susulan (§7.2) | Peta + laporan + slide |
| **H-1 Pertemuan 2**, pukul 23.59 | – | Unggah data, laporan, slide, `L{n}_ap.csv`, dan `L{n}_ringkasan.txt` | – |
| **Pertemuan 2** (minggu depan, 150 menit) | Tatap muka | Presentasi 3 kelompok + pleno lintas lantai (§7.3) | Kesimpulan kondisi gedung |

---

## 1. Capaian Pembelajaran

| Kode | Setelah praktikum, mahasiswa mampu… | Bukti |
|---|---|---|
| CP-1 | Menjelaskan parameter kualitas sinyal 5 GHz: RSSI, SNR, kanal, lebar kanal, CCI, utilisasi kanal | Soal pendahuluan, laporan |
| CP-2 | Menggambar denah berskala dan merancang titik ukur survei | Denah + `titik_L{n}.csv` |
| CP-3 | Melakukan survei pasif dan aktif dengan metodologi yang dapat diulang | Dataset mentah |
| CP-4 | Menghasilkan peta kualitas sinyal (heatmap + peta status) dan memperkirakan posisi AP | Peta keluaran `analyze.py` |
| CP-5 | Mengidentifikasi spot lemah beserta akar masalahnya | Tabel area lemah + analisis |
| CP-6 | Menyusun dan mempresentasikan rekomendasi penempatan dan konfigurasi AP berbasis data | Laporan + presentasi |

---

## 2. Kelompok dan Peran

Satu orang boleh memegang lebih dari satu peran jika anggota kelompok sedikit.

| Peran | Jumlah | Pertemuan 1 | Selama seminggu |
|---|---|---|---|
| **Koordinator** | 1 | Mengatur waktu dan zona, berkoordinasi dengan asisten, menjaga etika survei | Membagi pekerjaan, mengecek tenggat |
| **Juru denah** | 2 | Mengukur dan membuat sketsa denah, menandai titik ukur dan AP fisik, memotret AP | Digitalisasi denah + `mark_points.py` |
| **Surveyor Tim A / Tim B** | 2 + 2 | Survei pasif zona A / zona B (laptop + ponsel) | Survei susulan bila ada data yang kurang |
| **Operator uji aktif** | 1–2 | Ping/iperf3 di titik terpilih + uji roaming | Mengolah hasil uji aktif dan roaming |
| **Analis data** | 1–2 | Backup dan pengecekan kelengkapan data | Menjalankan `analyze.py` dan interpretasi |
| **Penulis / presenter** | 1–2 | Mencatat kondisi lapangan | Menyusun laporan dan slide, presentasi |

> **Zona A/B:** lantai dibagi dua, misalnya sayap kiri dan sayap kanan dari tangga utama, supaya dua tim bisa survei secara paralel.

---

## 3. Persiapan Sebelum Pertemuan 1

### 3.1 Perangkat per kelompok

- **Minimal 2 laptop** (satu per tim zona) dengan Wi-Fi yang mendukung **5 GHz**. Laptop boleh **Windows 10/11** atau **Linux** (termasuk Ubuntu Live USB).
- **Minimal 2 ponsel Android** dengan aplikasi **WiFiAnalyzer**. iPhone tidak bisa dipakai karena iOS tidak mengizinkan aplikasi melakukan scan Wi-Fi.
- Meteran atau laser distance meter (bila ada), papan jalan, kertas A3/milimeter blok, dan pensil.
- Buat folder kelompok `kelompok{k}_L{n}/` di setiap laptop, lalu **salin seluruh isi `scripts_survey_wifi/` ke dalamnya**, beserta subfolder `data/`, `denah/`, `raw/`, dan `output/` (Lampiran A). **Semua perintah di modul ini dijalankan dari folder kelompok tersebut.**

### 3.2 Instalasi: Windows

1. Pasang **Python 3.10+** dari python.org dan centang **"Add python.exe to PATH"**.
2. Buka *Command Prompt*, lalu jalankan:
   ```bat
   pip install pandas numpy scipy matplotlib
   ```
3. **Windows 11 (versi 24H2 ke atas):** aktifkan *Settings → Privacy & security → Location*, termasuk **"Let desktop apps access your location"**. Tanpa izin ini, Windows menolak memberi hasil scan Wi-Fi.
4. (Opsional, untuk uji throughput) unduh **iperf3 untuk Windows**, lalu letakkan `iperf3.exe` di folder skrip.
5. Uji scan (tanpa perlu hak admin):
   ```bat
   cd kelompok1_L2
   python scan_point.py --titik UJI --slot T0 --device laptopA --out uji.csv --n 1 --ssid "<SSID_KAMPUS>"
   ```
   Keluaran yang benar menampilkan jumlah BSSID, termasuk yang 5 GHz, beserta RSSI SSID kampus dalam **dBm**.

> Pada Windows, skrip memakai **Native Wi-Fi API** bawaan Windows (`wlan_win.py`), bukan `netsh`. Dengan cara ini RSSI terbaca dalam dBm yang sebenarnya (bukan persen), scan baru dipaksa di setiap pengukuran, dan informasi beacon (lebar kanal, BSS Load, keamanan) ikut terbaca.

### 3.3 Instalasi: Linux (Ubuntu)

```bash
sudo apt update
sudo apt install -y iw iperf3 python3-pip python3-venv
python3 -m venv ~/wifi && source ~/wifi/bin/activate
pip install pandas numpy scipy matplotlib

iw dev                                 # nama interface, mis. wlan0 / wlp2s0
iw list | grep -A 30 "Band 2"          # Band 2 = 5 GHz; pastikan ada frekuensi 5xxx MHz
cd kelompok2_L3
sudo ~/wifi/bin/python scan_point.py --iface wlan0 --titik UJI --slot T0 --device laptopA \
     --out uji.csv --n 1 --ssid "<SSID_KAMPUS>"
```

### 3.4 Ponsel Android

Pasang **WiFiAnalyzer** (VREM Software, open source, tersedia di F-Droid/Play Store). Matikan **Wi-Fi scan throttling** di *Developer options*. Tanpa langkah ini, Android membatasi scan menjadi 4 kali per 2 menit.

### 3.5 Soal pendahuluan (individu, dikumpulkan di awal Pertemuan 1)

1. Hitung *free space path loss* pada jarak 10 m untuk frekuensi 2437 MHz dan 5180 MHz. Berapa selisihnya, dan apa artinya bagi cakupan AP 5 GHz?
2. Sebutkan pembagian kanal 5 GHz (UNII-1, UNII-2, UNII-2e, UNII-3). Kanal mana yang termasuk **DFS**? Kanal mana yang diizinkan di Indonesia menurut regulasi Kominfo/Komdigi terbaru? (Cantumkan sumbernya.)
3. Jika semua AP memakai lebar kanal **80 MHz** dan hanya kanal non-DFS yang dipakai, berapa blok kanal 80 MHz yang tidak saling tumpang tindih? Apa dampaknya pada gedung dengan banyak AP?
4. Apa perbedaan **RSSI**, **SNR**, dan **channel utilization**? Mengapa RSSI yang tinggi tidak menjamin throughput yang tinggi?
5. Apa itu *co-channel interference* dan *sticky client*? Sebutkan standar 802.11 yang membantu proses roaming.
6. Mengapa rata-rata beberapa sampel RSSI sebaiknya dihitung dalam domain linear (mW), bukan langsung dirata-ratakan dalam dBm?

### 3.6 Persiapan oleh dosen/asisten

- Izin survei dari pimpinan fakultas dan unit TIK FILKOM, serta surat tugas.
- Informasi **SSID resmi yang dianalisis**, **IP gateway/target ping**, dan (bila tersedia) **server iperf3 berkabel**.
- Penetapan **titik acuan (0,0)** yang sama untuk ketiga lantai (mis. sudut tangga/lift utama) dan batas zona A/B setiap lantai.
- Folder bersama (Google Drive) untuk data ketiga kelompok.

---

## 4. Dasar Teori dan Kriteria Kualitas (5 GHz)

### 4.1 Karakteristik band 5 GHz

| Aspek | Keterangan |
|---|---|
| Pembagian kanal | UNII-1: 36–48 · UNII-2: 52–64 (DFS) · UNII-2e: 100–144 (DFS) · UNII-3: 149–165. Verifikasi kanal yang diizinkan di Indonesia (Soal 2) |
| Lebar kanal | 20 / 40 / 80 / 160 MHz. Semakin lebar, semakin cepat per klien, tetapi semakin **sedikit kanal yang tidak saling tumpang tindih** |
| Propagasi | FSPL 5 GHz ± **6,5 dB lebih besar** daripada 2,4 GHz pada jarak yang sama, dan redaman oleh dinding juga lebih besar, sehingga **cakupan per AP lebih kecil** |
| Keunggulan | Kanal lebih banyak, interferensi non-Wi-Fi lebih sedikit, dan throughput lebih tinggi |

**Free Space Path Loss:**  FSPL(dB) = 20·log₁₀(d_km) + 20·log₁₀(f_MHz) + 32,44

**Model log-distance (indoor):**  RSSI(d) = RSSI(d₀) − 10·n·log₁₀(d/d₀) − ΣL_dinding, dengan n ≈ 2 (ruang bebas) sampai 3,5–4 (gedung berdinding banyak).

### 4.2 Redaman material pada 5 GHz (nilai indikatif, sangat bervariasi)

| Material | Redaman (dB) |
|---|---|
| Partisi gipsum / kayu tipis | 3–5 |
| Kaca bening biasa | 3–8 (kaca low-E/berlapis logam bisa > 20) |
| Pintu kayu | 4–6 |
| Dinding bata | 8–15 |
| Dinding beton | 15–25 |
| Pelat lantai beton bertulang | 20–30+ |
| Lemari / pintu logam, lift | > 20 |

### 4.3 Hubungan RSSI dan data rate

Tabel berikut berisi **sensitivitas minimum** menurut standar 802.11ac/ax untuk 1 spatial stream. Kartu Wi-Fi nyata umumnya beberapa dB lebih baik.

| MCS | Modulasi | 20 MHz | 40 MHz | 80 MHz |
|---|---|---|---|---|
| 0 | BPSK 1/2 | −82 dBm | −79 | −76 |
| 4 | 16-QAM 3/4 | −70 | −67 | −64 |
| 7 | 64-QAM 5/6 | −64 | −61 | −58 |
| 9 | 256-QAM 5/6 | −57 | −54 | −51 |

Artinya, dengan **80 MHz**, sinyal −70 dBm hanya mendukung MCS rendah. Lebar kanal yang besar menuntut sinyal yang lebih kuat.

### 4.4 Satu SSID untuk seluruh gedung: identitas AP = BSSID

Di FILKOM, **semua AP memancarkan SSID yang sama**. Semua AP itu membentuk satu jaringan (satu *ESS*), dan klien bebas berpindah (roaming) dari satu AP ke AP lain, **termasuk ke AP di lantai lain**. Konsekuensinya untuk survei:

- **SSID tidak bisa dipakai untuk membedakan AP.** Setiap AP (tepatnya setiap radio) dikenali dari **BSSID**-nya (alamat MAC radio, mis. `a4:5e:60:xx:xx:xx`). Semua analisis di modul ini berbasis BSSID.
- Nama SSID tidak menunjukkan lantai. Lantai asal sebuah AP harus **disimpulkan dari data**: AP terdengar paling kuat di lantai mana (§9.5).
- Laptop di lantai 4 bisa saja tersambung ke AP lantai 3 yang sinyalnya "cukup". Ini menjadi salah satu penyebab kualitas buruk yang dicari dalam praktikum ini.
- **BSSID wajib selalu dicatat**, termasuk pada jalur darurat WiFiAnalyzer.

### 4.5 Kriteria status kualitas titik (dipakai oleh `analyze.py`)

| Parameter | BAIK | PERHATIAN | BURUK | KRITIS |
|---|---|---|---|---|
| RSSI AP terkuat (SSID kampus) | ≥ −67 dBm | −70 s.d. −67 | −80 s.d. −70 | < −80 atau tidak terdeteksi (**blank spot**) |
| AP cadangan (terkuat ke-2) | ≥ −75 dBm | < −75 (roaming berisiko) | | |
| Co-channel interference (radio lain di kanal overlap, ≥ −85 dBm) | ≤ 2 | > 2 | | |
| SNR (jika noise tersedia, Linux) | ≥ 25 dB | 15–25 | < 15 | |
| Utilisasi kanal (BSS Load) | ≤ 50% | > 50% | | |
| Throughput downlink (uji aktif) | | | < 10 Mbps | |
| Packet loss / RTT (uji aktif) | | RTT > 50 ms | loss > 2% | |

**Status akhir = kondisi terburuk** dari semua parameter. Titik BURUK/KRITIS yang saling berdekatan (≤ 8 m) dikelompokkan menjadi satu **area lemah** (A1, A2, …).

> Ambang di atas mengacu pada praktik umum desain WLAN enterprise (untuk data + voice/video). Kelompok boleh mendiskusikan apakah ambang ini sesuai untuk kebutuhan FILKOM, misalnya ruang kuliah vs koridor.

---

## 5. Tools

### 5.1 Apakah NetSpot bisa dipakai?

NetSpot (macOS/Windows) **bukan open source**. Menurut dokumentasi resminya:

| Edisi | Mode | Survei/heatmap | Batas | Ekspor laporan |
|---|---|---|---|---|
| **Free** | Inspector saja (scan AP) | **Tidak ada** | – | Tidak ada |
| PRO (berbayar) | Inspector, Planning, Survey | Ada | 50 zona/proyek, 50 snapshot/zona, 500 titik/snapshot | Ada |
| Enterprise (berbayar) | Inspector, Planning, Survey | Ada | Tanpa batas | Ada |

Artinya, **NetSpot Free tidak dapat membuat peta kualitas sinyal**. Karena itu, dalam praktikum ini:
- **Data dan peta utama wajib dihasilkan dengan toolchain di bawah.**
- NetSpot Free boleh dipakai untuk **validasi silang** (Inspector) di titik kalibrasi dan beberapa titik lain.
- Jika tersedia lisensi NetSpot PRO, heatmap NetSpot dapat dibandingkan dengan hasil `analyze.py` sebagai **nilai tambah**.

### 5.2 Toolchain

| Fungsi | Windows | Linux | Lisensi |
|---|---|---|---|
| Menggambar denah | draw.io (diagrams.net), Inkscape | sama | Apache-2.0 / GPL |
| Scan AP 5 GHz | `scan_point.py` (Native Wi-Fi API) | `scan_point.py` (`iw`) | skrip modul |
| Validasi silang | WiFiAnalyzer (Android); NetSpot Free (opsional) | WiFiAnalyzer | GPL-3.0 / freeware |
| Uji aktif | `active_point.py` + `ping` (+ `iperf3`) | sama | skrip modul / BSD |
| Log roaming | `roam_log.py` | sama | skrip modul |
| Analisis dan peta | Python + `analyze.py`, `gabung_lantai.py` | sama | skrip modul |

**Isi folder `scripts_survey_wifi/`:**

| Skrip | Fungsi |
|---|---|
| `scan_point.py` | Passive scan di satu titik (3× scan) → `passive_L{n}.csv`. Dengan `--ssid`, RSSI terkuat SSID kampus langsung ditampilkan |
| `wlan_win.py` | Modul pendukung untuk Windows (dipakai otomatis, tidak dijalankan langsung) |
| `active_point.py` | Info link, ping, dan iperf3 di satu titik → `active_L{n}.csv` |
| `roam_log.py` | Mencatat BSSID dan sinyal setiap detik selama uji roaming |
| `mark_points.py` | Kalibrasi skala denah digital dan penandaan titik ukur dengan klik → `titik_L{n}.csv` |
| `analyze.py` | Satu perintah → heatmap, peta status, area lemah, estimasi posisi AP, distribusi kanal, ringkasan |
| `gabung_lantai.py` | Menggabungkan tiga lantai: lantai asal AP dan *floor bleed* |

**Perbedaan data Windows dan Linux:** Windows tidak menyediakan *noise floor*, jadi kolom SNR kosong dan kriteria SNR dilewati. Semua metrik lain tetap sama. Kelompok wajib menyebutkan OS dan model kartu Wi-Fi yang dipakai di laporan.

---

## 6. Etika, Keamanan, dan K3

1. Survei **hanya bersifat pasif** dan uji kinerja dilakukan pada jaringan tempat mahasiswa memang berhak login.
2. **DILARANG:** deauthentication/jamming, membuat rogue AP/evil twin, cracking, menangkap payload lalu lintas pengguna lain, dan flooding. **Pelanggaran = nilai 0 untuk seluruh kelompok** dan dapat diproses sesuai peraturan akademik.
3. iperf3 maksimal **8 detik per arah per titik**. Jangan menjalankan iperf3 bersamaan dengan kelompok lain pada AP yang sama.
4. Jangan membuka plafon atau menyentuh perangkat jaringan. Cukup observasi visual dan foto.
5. Masuk ke ruang kelas/lab hanya jika ruangan tidak dipakai atau sudah ada izin. Jangan menghalangi jalur evakuasi.
6. Bawa surat tugas dan jelaskan kegiatan dengan sopan bila ditanya.

---

## 7. Alur Kegiatan

### 7.1 Pertemuan 1: Eksperimen lapangan (150 menit)

| Menit | Tahap | Kegiatan | Hasil |
|---|---|---|---|
| 0–15 | **Briefing** | Kumpulkan soal pendahuluan, cek laptop (uji scan), bagi peran dan zona | Semua laptop lulus uji scan |
| 15–45 | **Tahap 1: Sketsa denah** | Ukur dan buat sketsa denah di kertas, tandai AP fisik, tentukan dan beri nomor titik ukur (§8.1) | Sketsa bernomor + foto AP |
| 45–55 | **Tahap 2: Kalibrasi** | Semua perangkat melakukan scan di titik `L{n}-000` (§8.2) | Offset perangkat |
| 55–115 | **Tahap 3: Survei pasif** | Tim A dan B menyurvei semua titik di zonanya (§8.3) | `passive_L{n}_A/B.csv` + RSSI tertulis di sketsa |
| 115–140 | **Tahap 4: Survei aktif + roaming** | 6–8 titik terpilih + 1 lintasan koridor (§8.4) | `active_L{n}.csv`, log roaming |
| 140–150 | **Penutup** | Backup ke folder bersama, cek kelengkapan data, bagi tugas seminggu | Data aman |

### 7.2 Selama seminggu (tugas terstruktur)

1. **Digitalisasi denah** dari sketsa (§9.1) → `denah_L{n}.png`.
2. **Koordinat titik** dengan `mark_points.py`, menggunakan **nomor titik yang sama dengan sketsa** → `titik_L{n}.csv`, lalu buat `ap_fisik_L{n}.csv`.
3. **Analisis** dengan `analyze.py` (§9.2–9.4).
4. **Survei susulan (opsional, nilai tambah):** melengkapi titik yang terlewat, atau mengulang 10 titik di **jam berbeda** (pagi vs jam sibuk) untuk melihat pengaruh beban.
5. **Laporan dan slide** (§10).
6. **Unggah** paling lambat H-1 Pertemuan 2 pukul 23.59.

### 7.3 Pertemuan 2: Presentasi dan pleno (150 menit)

| Menit | Kegiatan |
|---|---|
| 0–10 | Pembukaan, urutan presentasi |
| 10–85 | Presentasi 3 kelompok × 25 menit (15 menit presentasi + 10 menit tanya jawab) |
| 85–125 | **Pleno lintas lantai:** asisten menampilkan hasil `gabung_lantai.py`, diskusi pertanyaan §9.5 |
| 125–150 | Rumusan rekomendasi tingkat gedung, umpan balik dosen, refleksi |

---

## 8. Prosedur Lapangan (Pertemuan 1)

### 8.1 Tahap 1: Sketsa denah dan titik ukur

**Sketsa (di kertas A3/milimeter blok, pensil):**
- Gambar dinding luar, koridor, ruang (tulis nomor/nama ruang), pintu, tangga, lift, dan toilet.
- **Ukur dan tulis angka dimensi** pada sketsa: panjang dan lebar koridor serta lebar setiap ruang. Gunakan meteran/laser, atau **hitung ubin lantai** (ukur satu ubin, mis. 60 × 60 cm), atau langkah kaki yang sudah dikalibrasi.
- Tandai **titik acuan (0,0)** yang ditetapkan asisten dan panah arah utara.
- Beri keterangan material dinding penting: beton, bata, kaca, partisi gipsum, pintu/lemari logam.
- Tandai **setiap AP yang terlihat** (plafon/dinding) dengan ■ dan kode `AP-L{n}-01`, `AP-L{n}-02`, …, lalu **foto** setiap AP. Jika label atau model AP terbaca dari bawah, catat juga.

**Titik ukur (target 25–35 titik per lantai), gambar di sketsa dan beri nomor `L{n}-001`, `L{n}-002`, …:**
- Koridor: 1 titik setiap **± 5 m**.
- Setiap ruang yang bisa diakses: minimal **1 titik di tengah**. Ruang besar: tambah 1 titik di **pojok terjauh dari AP**.
- Area khusus: tangga, depan lift, lounge mahasiswa, ujung koridor, dan dekat jendela/balkon.
- 1 **titik kalibrasi** `L{n}-000` di area terbuka dekat AP.
- **Foto sketsa** setelah selesai dan unggah ke folder bersama. Foto ini menjadi acuan digitalisasi.

### 8.2 Tahap 2: Kalibrasi

Di titik `L{n}-000`, semua laptop melakukan 5 scan bersamaan, dan ponsel mencatat RSSI SSID kampus dari WiFiAnalyzer:

```bat
:: Windows
python scan_point.py --titik L3-000 --lantai 3 --slot T1 --device laptopA --out data\passive_L3_A.csv --n 5 --ssid "<SSID_KAMPUS>"
```
```bash
# Linux
sudo ~/wifi/bin/python scan_point.py --iface wlan0 --titik L3-000 --lantai 3 --slot T1 \
     --device laptopA --out data/passive_L3_A.csv --n 5 --ssid "<SSID_KAMPUS>"
```

Catat selisih RSSI antar-perangkat untuk AP yang sama (**offset perangkat**) di Lembar Kerja. Offset ini dibahas di laporan dan **tidak** dipakai untuk mengoreksi data secara otomatis.

### 8.3 Tahap 3: Survei pasif

> Koordinat titik **belum diperlukan** saat survei. Cukup nomor titik dari sketsa. Koordinat disambungkan saat analisis (`analyze.py --titik`).

Di setiap titik:
1. Berdiri tepat di titik sesuai sketsa. Pegang laptop setinggi **± 1 m** dengan layar menghadap **arah yang sama** di semua titik. Surveyor tidak berdiri di antara laptop dan arah AP terdekat.
2. Jalankan (3 scan, ± 20–25 detik):
   ```bat
   :: Windows
   python scan_point.py --titik L3-017 --lantai 3 --slot T1 --device laptopA --out data\passive_L3_A.csv --ssid "<SSID_KAMPUS>" --catatan "kelas kosong, pintu tertutup"
   ```
   ```bash
   # Linux
   sudo ~/wifi/bin/python scan_point.py --iface wlan0 --titik L3-017 --lantai 3 --slot T1 \
        --device laptopA --out data/passive_L3_A.csv --ssid "<SSID_KAMPUS>" --catatan "kelas kosong"
   ```
3. **Tulis RSSI terkuat SSID kampus** yang tampil di layar ke sketsa, di samping nomor titik. Angka ini dipakai untuk memilih titik uji aktif (§8.4) dan membantu presentasi.
4. Secara bersamaan, anggota lain mengambil screenshot **WiFiAnalyzer** (tab *Access Points*, filter 5 GHz) dengan nama file `L3-017.png`.
5. Isi Lembar Kerja: centang titik, perkiraan jumlah orang, dan pintu terbuka/tertutup.
6. Pindah ke titik berikutnya. Target: ± 2 menit per titik.

**Jalur darurat (jika laptop bermasalah):** catat manual dari WiFiAnalyzer ke CSV dengan kolom `titik_id,ssid,bssid,channel,rssi_dbm`, **satu baris untuk setiap BSSID 5 GHz yang terlihat** (bukan hanya yang terkuat). Karena SSID-nya sama semua, BSSID adalah satu-satunya pembeda AP. `analyze.py` tetap bisa memprosesnya.

### 8.4 Tahap 4: Survei aktif dan roaming

**Pilih 6–8 titik** dari RSSI yang tertulis di sketsa:
- 3 titik dengan **RSSI terendah atau tidak terdeteksi**,
- 2 titik dengan **RSSI terbaik** (pembanding),
- 1–3 titik yang **RSSI-nya bagus tetapi dicurigai**, misalnya area padat mahasiswa atau banyak AP terdengar.

Hubungkan laptop ke **SSID kampus**, lalu di setiap titik jalankan:
```bat
:: Windows (target = IP gateway; lihat "Default Gateway" pada perintah ipconfig)
python active_point.py --titik L3-017 --target <IP_GATEWAY> --out data\active_L3.csv [--iperf <IP_SERVER>]
```
```bash
# Linux (ip route | grep default -> IP gateway)
~/wifi/bin/python active_point.py --iface wlan0 --titik L3-017 --target <IP_GATEWAY> \
     --out data/active_L3.csv [--iperf <IP_SERVER>]
```

**Uji roaming (1 lintasan koridor, ± 5 menit):** berjalan pelan dari ujung ke ujung koridor sambil menjalankan dua terminal:
```bat
:: Terminal 1 (Windows) - BSSID dan sinyal setiap detik
python roam_log.py --out raw\roaming_L3.csv
:: Terminal 2 (PowerShell) - ping dengan cap waktu
ping -t <IP_GATEWAY> | ForEach-Object { "{0} {1}" -f (Get-Date -Format HH:mm:ss), $_ } | Tee-Object raw\roaming_ping_L3.txt
```
```bash
# Linux
python3 roam_log.py --iface wlan0 --out raw/roaming_L3.csv                          # terminal 1
ping -i 0.2 <IP_GATEWAY> | while read l; do echo "$(date +%T) $l"; done | tee raw/roaming_ping_L3.txt   # terminal 2
```
Tandai di sketsa lokasi saat log menampilkan **"PINDAH AP"**. Catat **sinyal sesaat sebelum pindah** dan **ping yang hilang** di sekitar waktu tersebut. Jika klien tetap menempel ke AP jauh sampai sinyal < −75 dBm, itu tanda **sticky client** atau kurangnya overlap antar-AP. Karena SSID-nya sama di semua lantai, perhatikan juga apakah klien **berpindah ke AP lantai lain**. Cocokkan BSSID di log dengan daftar AP per lantai hasil pleno (§9.5).

> Pada Windows, sinyal di `roam_log.py` adalah **aproksimasi** dari persentase kualitas sinyal ((persen/2) − 100), karena pembacaan dBm yang akurat memerlukan scan yang akan mengganggu uji roaming. Sebutkan hal ini di laporan.

### 8.5 Penutup

Gabungkan data tim A dan B, lalu unggah seluruh folder ke folder bersama:
```bash
# Linux/macOS
head -1 data/passive_L3_A.csv > data/passive_L3.csv
tail -q -n +2 data/passive_L3_A.csv data/passive_L3_B.csv >> data/passive_L3.csv
```
```bat
:: Windows (PowerShell)
Import-Csv data\passive_L3_A.csv, data\passive_L3_B.csv -Encoding UTF8 | Export-Csv data\passive_L3.csv -NoTypeInformation -Encoding UTF8
```

---

## 9. Pengolahan dan Analisis (selama seminggu)

### 9.1 Digitalisasi denah

**Tool:** draw.io/diagrams.net (disarankan), Inkscape, atau LibreCAD.

| Aturan | Ketentuan |
|---|---|
| Skala | **1 meter = 20 piksel**. Di draw.io, aktifkan grid 20 px, sehingga 1 kotak = 1 m |
| Titik acuan (0,0) | Diletakkan pada koordinat piksel **yang sama di ketiga denah**, mis. (100, 100), supaya ketiga lantai bisa ditumpuk |
| Ukuran kanvas | **Sama** untuk ketiga lantai (disepakati ketiga kelompok) |
| Isi | Dinding, ruang + nomor, pintu, tangga, lift, toilet, material penting, AP fisik (■ + kode), panah utara |
| Ekspor | `denah_L{n}.png` tanpa margin + file sumber `.drawio`/`.svg` |

Setelah itu:
```bash
python mark_points.py --denah denah/denah_L3.png --lantai 3 --out data/titik_L3.csv
```
Klik dua ujung koridor dan masukkan panjang sebenarnya. Skala yang tercetak harus **≈ 20 px/m**; jika jauh berbeda, periksa kembali denahnya. Lalu klik titik ukur **berurutan sesuai nomor di sketsa**. Buat juga `ap_fisik_L{n}.csv`:
```csv
ap_id,x_px,y_px,keterangan
AP-L3-01,340,180,plafon koridor depan R.3.x
```

### 9.2 Menjalankan analisis

```bash
python analyze.py --passive data/passive_L3.csv --titik data/titik_L3.csv --ssid "<SSID_KAMPUS>" \
       --denah denah/denah_L3.png --scale 20 --out output/L3 \
       --active data/active_L3.csv --ap-fisik data/ap_fisik_L3.csv
```

| File keluaran | Isi | Dipakai untuk |
|---|---|---|
| `L3_heatmap.png` | Heatmap RSSI AP terkuat + garis kontur −67 dBm (hijau) dan −75 dBm (merah) | Peta cakupan |
| `L3_status.png` | Titik berwarna menurut status, area lemah A1, A2, … (arsir merah), AP estimasi (▲ penuh = di lantai ini, △ kosong = mungkin tembus dari lantai lain), dan AP fisik (□) | **Peta Kualitas Sinyal dan Penempatan AP** |
| `L3_titik.csv` | Metrik per titik: rssi1, rssi2, kanal, lebar, CCI, SNR, util, status, penyebab | Tabel lampiran |
| `L3_area_lemah.csv` | Daftar area lemah + penyebab dominan | Inti analisis |
| `L3_ap.csv` | Estimasi posisi setiap radio AP (weighted centroid dari 3 titik terkuat), kanal, keyakinan (tinggi: RSSI maks ≥ −55 dBm; sedang: −65 s.d. −55; rendah: < −65) | Pemetaan AP |
| `L3_kanal.png` | Jumlah radio per kanal 5 GHz (SSID kampus vs lainnya) | Analisis channel plan |
| `L3_ringkasan.txt` | Persentase cakupan dan jumlah status | Angka kunci laporan |

Contoh keluaran dari **data sintetis** (bukan data FILKOM) ada di `scripts_survey_wifi/contoh_output/`.

> **Keterbatasan yang wajib disadari:**
> (a) Interpolasi heatmap hanya valid **di dalam** sebaran titik ukur. Jika semua titik berada pada satu garis (mis. hanya koridor), skrip otomatis beralih ke interpolasi IDW dengan radius 5 m dari setiap titik, dan metodenya tercantum di judul heatmap.
> (b) Estimasi posisi AP adalah **perkiraan**. Keyakinan "sedang" (△) berarti AP kemungkinan tembus dari lantai atas/bawah atau berada di ruang tertutup. Keyakinan "rendah" tidak digambar, tetapi tetap tercantum di `L{n}_ap.csv`. Konfirmasi lantai asal AP saat pleno (`gabung_lantai.py`).
> (c) Setiap BSSID dihitung sebagai satu radio. BSSID yang hanya beda digit terakhir **baru digabung jika memancarkan SSID berbeda** (satu radio, multi-SSID). Dengan SSID yang sama di seluruh FILKOM, BSSID praktis tidak digabung, jadi AP ber-MAC berurutan tetap terhitung terpisah. Jika penggabungan terjadi, skrip menampilkan baris `Info:`.
> (d) Survei dilakukan pada satu rentang waktu. Kondisi beban pada jam lain bisa berbeda.

### 9.3 Langkah interpretasi

1. **Cakupan:** berapa % titik yang BAIK (≥ −67 dBm)? Di ruang mana blank spot berada?
2. **Pemetaan AP:** bandingkan ▲ (estimasi) dengan □ (fisik).
   - AP fisik tanpa ▲ di dekatnya → AP mati, memakai SSID lain, atau hanya 2,4 GHz.
   - ▲ tanpa □ → AP tersembunyi di dalam ruangan atau berasal dari lantai lain.
3. **Kanal:** apakah AP bertetangga memakai kanal yang sama? Berapa lebar kanalnya? Apakah kanal DFS dimanfaatkan?
4. **Area lemah:** untuk setiap area A1, A2, …, tentukan akar masalah dengan tabel §9.4.
5. **Verifikasi aktif:** apakah titik yang statusnya buruk memang menunjukkan throughput rendah, loss, atau RTT tinggi? Apakah ada titik "RSSI bagus tetapi lambat"?
6. **Roaming:** di mana perpindahan AP terjadi, dan apakah ada ping yang hilang?

### 9.4 Tabel diagnosis: gejala → akar masalah → rekomendasi

| Gejala di data | Kemungkinan akar masalah | Contoh rekomendasi |
|---|---|---|
| RSSI < −75, AP terdekat jauh (> 15–20 m) | **Coverage gap**: jumlah AP kurang | Tambah AP, atau pindahkan AP ke titik tengah area |
| RSSI rendah padahal AP dekat, terhalang dinding beton/ruang tertutup | **Atenuasi material**: AP di koridor melayani ruang kelas | Tempatkan AP di dalam ruang kelas; perhatikan orientasi AP |
| RSSI baik, tetapi throughput rendah atau loss tinggi, CCI > 2 | **Co-channel interference** | Susun ulang kanal, turunkan lebar kanal ke 40/20 MHz, manfaatkan kanal DFS, atur daya pancar |
| RSSI baik, utilisasi kanal > 50%, banyak stasiun | **Beban/kepadatan** | Tambah AP kapasitas (daya rendah), band steering, batasi data rate rendah |
| AP terkuat berasal dari lantai lain | **Floor bleed**: desain vertikal tidak diperhitungkan | Kurangi daya, atur penggunaan kanal secara vertikal, tambah AP di lantai sendiri |
| Tidak ada AP cadangan ≥ −75 dBm, roaming disertai ping hilang | **Overlap antar-AP kurang** | Rapatkan jarak AP (overlap sel 15–20%), aktifkan 802.11k/v/r |
| SNR rendah padahal RSSI cukup | **Noise/interferensi non-Wi-Fi** | Identifikasi sumber interferensi, pindahkan kanal |

### 9.5 Analisis lintas lantai (Pertemuan 2)

Setelah ketiga kelompok mengunggah `L{n}_ap.csv` dan `L{n}_ringkasan.txt`, asisten menjalankan:
```bash
python gabung_lantai.py --ap 2:output/L2_ap.csv 3:output/L3_ap.csv 4:output/L4_ap.csv \
       --titik 2:output/L2_titik.csv 3:output/L3_titik.csv 4:output/L4_titik.csv \
       --ringkasan output/L2_ringkasan.txt output/L3_ringkasan.txt output/L4_ringkasan.txt \
       --out output/gedung
```
Hasilnya menunjukkan:
- **Lantai asal setiap AP** (berdasarkan BSSID), sehingga daftar AP per lantai bisa disusun meskipun SSID-nya sama.
- **Titik yang AP terkuatnya berasal dari lantai lain** (`gedung_titik_lantai_lain.csv`). Di titik ini, klien kemungkinan besar tersambung ke AP lantai atas/bawah. Ini indikasi kuat kurangnya AP di lantai tersebut.
- Selisih RSSI antar-lantai, yang menjadi **estimasi kasar atenuasi pelat lantai**.

 Selisih ini juga dipengaruhi jarak horizontal, jadi yang paling bermakna adalah titik yang berada tepat di atas atau di bawah AP. Hal ini menjadi bahan diskusi.

**Pertanyaan pleno:**
1. Lantai mana yang kualitasnya paling baik dan paling buruk? Apakah penyebabnya sama?
2. Apakah ada AP yang dominan terdengar di lantai lain? Berapa estimasi redaman lantainya, dan apakah sesuai dengan nilai pustaka (§4.2)?
3. Apakah pola kanal di ketiga lantai sudah memperhitungkan AP yang bertumpuk vertikal?
4. Berapa titik di setiap lantai yang "dilayani" AP lantai lain? Apa dampaknya bagi pengguna (roaming, kapasitas AP lantai lain), dan apa solusinya?
5. Jika anggaran hanya cukup untuk **3 AP baru di seluruh gedung**, di mana AP tersebut dipasang?

---

## 10. Luaran dan Pengumpulan

| No | Luaran | Batas waktu |
|---|---|---|
| 1 | Soal pendahuluan (individu) | Awal Pertemuan 1 |
| 2 | Data mentah + foto sketsa + foto AP (folder bersama) | Akhir Pertemuan 1 |
| 3 | Folder data lengkap (struktur Lampiran A) | H-1 Pertemuan 2, 23.59 |
| 4 | **Laporan kelompok** (PDF, maks. 12 halaman + lampiran) | H-1 Pertemuan 2, 23.59 |
| 5 | Slide presentasi (maks. 12 slide) | H-1 Pertemuan 2, 23.59 |
| 6 | Penilaian sejawat (individu, formulir) | Setelah Pertemuan 2 |

### Isi slide (maks. 12)
1. Tim, lantai, dan metodologi singkat (perangkat, OS, jumlah titik)
2. Denah lantai dan titik ukur
3. Heatmap RSSI 5 GHz
4. **Peta Kualitas Sinyal dan Penempatan AP**
5. Pemetaan AP: estimasi vs fisik, kanal, dan lebar kanal
6. Tabel area lemah + akar masalah
7. Hasil uji aktif
8. Hasil uji roaming
9. Rekomendasi (prioritas 1–3), idealnya digambar di denah
10. Keterbatasan dan kesimpulan

### Template laporan

```markdown
# Laporan Survei Wi-Fi 5 GHz – Lantai {n} Gedung FILKOM – Kelompok {k}

## 1. Identitas
Anggota + peran | tanggal & jam survei | perangkat (model laptop, OS, kartu Wi-Fi, ponsel) | SSID yang dianalisis

## 2. Denah dan Metodologi
- Denah digital + skala + titik acuan; cara mengukur dimensi; jumlah titik
- Prosedur survei pasif/aktif/roaming; offset perangkat hasil kalibrasi

## 3. Hasil
### 3.1 Ringkasan angka
| Metrik | Nilai |
| % titik RSSI ≥ −67 dBm | |
| % titik RSSI < −80 dBm (blank spot) | |
| % titik dengan AP cadangan ≥ −75 dBm | |
| RSSI median / minimum | |
| Jumlah radio SSID kampus & kanal yang dipakai | |
| Lebar kanal yang dipakai | |
### 3.2 Heatmap RSSI (gambar + interpretasi)
### 3.3 Peta Kualitas Sinyal dan Penempatan AP (gambar + interpretasi)
### 3.4 Pemetaan AP: estimasi vs fisik (tabel ap_id / radio / kanal / keyakinan / cocok?)
### 3.5 Distribusi kanal
### 3.6 Uji aktif dan roaming (tabel + temuan)

## 4. Analisis Area Lemah
| Area | Lokasi (ruang) | Titik | RSSI rata | Status | Akar masalah (bukti data) |

## 5. Rekomendasi
| Prioritas | Area | Tindakan (lokasi/kanal/daya/lebar kanal) | Dasar data | Perkiraan dampak |

## 6. Keterbatasan dan Validasi
Offset perangkat, perbedaan Windows/Linux, keterbatasan interpolasi, heuristik radio, waktu survei, dll.

## 7. Kesimpulan

## Lampiran
L{n}_titik.csv (ringkas), foto sketsa, foto AP fisik, log roaming, lembar kerja
```

---

## 11. Rubrik Penilaian (Kelompok)

| Komponen | Bobot | Sangat Baik (≥ 85) | Cukup (65–75) | Kurang (< 55) |
|---|---|---|---|---|
| **Denah** | 15% | Sketsa terukur + denah digital berskala benar (≈ 20 px/m), lengkap (ruang, material, AP fisik), selaras titik acuan | Berskala, kurang lengkap | Tidak berskala / sketsa kasar |
| **Kualitas data** | 20% | ≥ 25 titik, 3 scan/titik, kalibrasi, validasi Android, format sesuai | Titik kurang atau tidak merata | Data tidak konsisten / tidak dapat diproses |
| **Peta dan pemetaan AP** | 20% | Heatmap + peta status benar; estimasi AP dibandingkan dengan AP fisik; keterbatasan dijelaskan | Peta ada, interpretasi dangkal | Peta salah atau tanpa interpretasi |
| **Analisis spot lemah** | 20% | Setiap area lemah punya akar masalah yang didukung data (RSSI, CCI, util, aktif, roaming) | Akar masalah umum | Hanya menyebut lokasi lemah |
| **Rekomendasi** | 10% | Spesifik (lokasi, kanal, lebar kanal, daya), diprioritaskan, realistis, digambar di denah | Umum ("tambah AP") | Tidak ada / tidak relevan |
| **Laporan dan presentasi** | 15% | Laporan rapi sesuai template; presentasi jelas, tepat waktu, menjawab pertanyaan | Cukup | Kurang |
| **Nilai tambah** | +5 | Survei jam berbeda (pengaruh beban), pembanding NetSpot/Linux vs Windows, analisis beacon (Wireshark/Kismet), atau analisis kanal DFS | | |

**Nilai individu** = nilai kelompok × faktor penilaian sejawat (0,8–1,1) + nilai soal pendahuluan (bobot ditentukan dosen).

**Keterlambatan unggah:** −10 poin per 12 jam.

---

## 12. Troubleshooting

| Masalah | Solusi |
|---|---|
| **Windows:** `akses ditolak ... Location` | Aktifkan *Settings → Privacy & security → Location* dan "Let desktop apps access your location", lalu ulangi |
| **Windows:** `'python' is not recognized` | Instal ulang Python dengan mencentang "Add python.exe to PATH", atau pakai `py` sebagai pengganti `python` |
| **Windows:** tidak ada BSSID 5 GHz | Cek *Device Manager → Network adapters → Properties → Advanced*, pastikan band 5 GHz tidak dinonaktifkan ("Preferred Band"/"Wireless Mode") |
| **Linux:** `iw scan` → *Operation not permitted* | Jalankan dengan `sudo` |
| **Linux:** `iw scan` → *Device or resource busy* | Tunggu beberapa detik lalu ulangi. Jika tetap gagal: `sudo nmcli radio wifi off && sudo nmcli radio wifi on` |
| Adaptor tidak mendukung 5 GHz | Beberapa adaptor USB murah hanya mendukung 2,4 GHz. Ganti laptop, atau pakai jalur darurat WiFiAnalyzer |
| Hasil scan sedikit/tidak lengkap | Ulangi scan. Scan pasif pada kanal DFS lebih lambat, jadi 3 scan per titik penting |
| Kolom `noise_dbm` kosong | Wajar di Windows dan pada sebagian driver Linux. Kriteria SNR dilewati |
| Kolom BSS Load kosong | AP tidak mengiklankan elemen BSS Load. Kriteria utilisasi dilewati |
| iperf3 *connection refused/timeout* | Server tidak tersedia atau diblokir. Cukup gunakan ping, lalu catat sebagai keterbatasan |
| `PERINGATAN: titik tanpa koordinat` | Ada nomor titik di data yang belum diklik di `mark_points.py`. Lengkapi `titik_L{n}.csv` |
| `ERROR: SSID '...' tidak ditemukan` | Nama SSID salah ketik (huruf besar/kecil berpengaruh). Skrip menampilkan daftar SSID yang terdeteksi; salin nama yang tepat |
| `ERROR: tidak ada data band 5 GHz` | Laptop hanya menangkap 2,4 GHz. Periksa dukungan 5 GHz pada adaptor (lihat baris di atas) |
| `ERROR: semua titik tidak punya koordinat` | Survei dilakukan tanpa koordinat (sesuai prosedur). Tambahkan `--titik data/titik_L{n}.csv` |
| `PERINGATAN: titik uji aktif tanpa data pasif` | Nomor titik di `active_L{n}.csv` tidak sama dengan nomor di survei pasif. Periksa salah ketik |
| Heatmap kosong di sebagian area | Tidak ada titik ukur di area itu, jadi tambah titik (survei susulan) |
| Posisi titik di peta meleset | Periksa skala denah dan pastikan `titik_L{n}.csv` dibuat dari denah yang sama |

---

## 13. Glosarium Singkat

| Istilah | Arti |
|---|---|
| **AP / radio / BSSID** | Satu AP bisa memiliki beberapa radio (mis. 2,4 dan 5 GHz); setiap radio punya BSSID sendiri. Di FILKOM semua AP memakai SSID yang sama, sehingga BSSID menjadi identitas AP |
| **ESS** | Extended Service Set: sekumpulan AP dengan SSID yang sama yang membentuk satu jaringan, tempat klien bisa roaming |
| **RSSI** | Kuat sinyal yang diterima (dBm, semakin mendekati 0 semakin kuat) |
| **SNR** | Selisih sinyal terhadap derau (dB) |
| **CCI** | Co-channel interference: beberapa radio di kanal yang sama/overlap berbagi airtime |
| **BSS Load** | Elemen beacon yang memuat jumlah stasiun dan utilisasi kanal menurut AP |
| **DFS** | Dynamic Frequency Selection: kanal 5 GHz yang wajib menghindari radar |
| **Blank spot** | Lokasi tanpa sinyal yang layak (< −80 dBm) |
| **Floor bleed** | Sinyal AP yang menembus ke lantai lain |
| **Sticky client** | Klien yang tidak mau pindah ke AP yang lebih kuat |
| **802.11k/v/r** | Standar pendukung roaming: laporan AP tetangga / BSS transition / fast transition |

---

## 14. Referensi

1. Coleman, D. & Westcott, D. *CWNA Certified Wireless Network Administrator Study Guide* (bab RF, Site Survey, WLAN Design).
2. IEEE Std 802.11-2020, *Wireless LAN MAC and PHY Specifications*.
3. Cisco, *Enterprise Mobility Design Guide* dan *High Density Experience (HDX) Deployment Guide*.
4. ITU-R P.1238, *Propagation data and prediction methods for indoor radiocommunication systems*.
5. Peraturan Kominfo/Komdigi terbaru tentang penggunaan spektrum frekuensi radio berdasarkan izin kelas (untuk alokasi 5 GHz di Indonesia).
6. Microsoft Learn, *Native Wifi API* (`WlanScan`, `WlanGetNetworkBssList`).
7. Dokumentasi: `iw` (wireless.wiki.kernel.org), iperf3, WiFiAnalyzer (github.com/VREMSoftwareDevelopment/WiFiAnalyzer), draw.io, NetSpot (netspotapp.com/help).

---

## Lampiran A. Struktur Folder Kelompok

```
kelompok{k}_L{n}/
├── *.py       salinan isi scripts_survey_wifi/ (perintah dijalankan dari sini)
├── denah/     sketsa_L{n}.jpg (foto), denah_L{n}.png, denah_L{n}.drawio, foto AP fisik
├── data/      titik_L{n}.csv, ap_fisik_L{n}.csv, passive_L{n}.csv (+ _A, _B), active_L{n}.csv
├── raw/       raw/active/*.txt|json, roaming_*.csv|txt, android/*.png
├── output/    L{n}_heatmap.png, L{n}_status.png, L{n}_*.csv, L{n}_ringkasan.txt
└── laporan/   laporan PDF + slide
```

**Kolom `passive_L{n}.csv`** (dihasilkan `scan_point.py`):
`timestamp, lantai, titik_id, x_px, y_px, slot_waktu, device, scan_ke, ssid, bssid, freq_mhz, channel, band, width_mhz, rssi_dbm, noise_dbm, sta_count, ch_util_pct, security, pmf, catatan`
(`x_px`/`y_px` boleh kosong saat survei, karena diisi dari `titik_L{n}.csv` saat analisis.)

**Kolom `active_L{n}.csv`** (dihasilkan `active_point.py`):
`timestamp, titik_id, bssid_assoc, freq_mhz, rssi_dbm, tx_rate_mbps, rtt_avg_ms, rtt_max_ms, jitter_ms, loss_pct, down_mbps, up_mbps`

## Lampiran B. Lembar Kerja Lapangan (cetak)

**Kalibrasi (titik L_-000)**

| Perangkat | Model / OS / kartu Wi-Fi | RSSI AP referensi (rata-rata 5 scan) | Offset terhadap laptop A |
|---|---|---|---|
| Laptop A | | | 0 |
| Laptop B | | | |
| Ponsel 1 | | | |

**Survei pasif – Zona ___  Tim ___**

| Titik | ✓ | Jam | RSSI SSID kampus (dBm) | Jumlah orang (±) | Pintu (B/T) | Catatan |
|---|---|---|---|---|---|---|
| L_-001 | | | | | | |
| L_-002 | | | | | | |
| … | | | | | | |

**Survei aktif**

| Titik | Alasan dipilih | BSSID | RSSI | RTT avg | Loss | Down/Up (Mbps) |
|---|---|---|---|---|---|---|
| | | | | | | |

**Roaming**

| Perpindahan ke- | Lokasi (dekat titik) | BSSID lama → baru | Sinyal sebelum pindah | Ping hilang |
|---|---|---|---|---|
| | | | | |

## Lampiran C. Checklist

**Sebelum Pertemuan 1**
- [ ] Python + library terpasang, `scan_point.py` lulus uji dan mendeteksi 5 GHz (min. 2 laptop)
- [ ] Windows 11: izin Location untuk desktop apps sudah aktif
- [ ] WiFiAnalyzer terpasang, scan throttling dimatikan
- [ ] Soal pendahuluan selesai

**Pertemuan 1**
- [ ] Sketsa denah terukur + titik bernomor + AP fisik (difoto)
- [ ] Kalibrasi di `L{n}-000`
- [ ] Semua titik disurvei, RSSI tertulis di sketsa
- [ ] 6–8 titik aktif + 1 lintasan roaming
- [ ] Data tim A + B digabung dan diunggah

**Selama seminggu**
- [ ] `denah_L{n}.png` (20 px/m, titik acuan sesuai kesepakatan)
- [ ] `titik_L{n}.csv` (nomor sama dengan sketsa) + `ap_fisik_L{n}.csv`
- [ ] `analyze.py` dijalankan, tidak ada peringatan titik tanpa koordinat
- [ ] Laporan + slide + `L{n}_ap.csv` + `L{n}_ringkasan.txt` diunggah H-1

---

*Catatan untuk dosen/asisten (hapus sebelum dibagikan):*
- *Isi: SSID resmi yang dianalisis, IP gateway/target ping, server iperf3 (jika ada), titik acuan (0,0) untuk lantai 2–4, ukuran kanvas denah bersama, dan batas zona A/B setiap lantai.*
- *Urus izin dan koordinasi dengan unit TIK FILKOM sebelum Pertemuan 1. Sebaiknya survei dilakukan pada jam kuliah normal agar kondisi beban realistis.*
- *Siapkan folder bersama untuk data ketiga kelompok, dan jalankan `gabung_lantai.py` sebelum Pertemuan 2.*
- *Status pengujian skrip:*
  - *Lulus di Python 3.8 (pandas 1.4) dan Python 3.10 (pandas 2.3, numpy 2.2, scipy 1.15, matplotlib 3.10); pyflakes bersih.*
  - *Uji end-to-end 3 lantai dengan `iw`/`ping`/`iperf3` tiruan: mark_points (klik tersimulasi) → scan tanpa koordinat → gabung A/B → uji aktif → analyze → gabung_lantai. Posisi AP terestimasi meleset < 1,3 m, lantai asal AP benar semua, dan floor bleed terdeteksi 20,2 dB (model 20 dB).*
  - *16 kasus sulit analyze.py lulus (titik segaris koridor, 1–2 titik, SSID salah/hilang, hanya 2,4 GHz, SSID emoji/koma, data manual minimal, koordinat kosong, dll.).*
  - *Jalur Windows diuji dengan tiruan wlanapi.dll yang menulis struktur ke memori sesuai layout Windows x64: scan, info link, ping Windows berbahasa Indonesia, log roaming, dan pesan saat izin Location ditolak.*
  - *Belum diuji: Windows dan `iw` sungguhan, serta jendela GUI `mark_points.py`. **Coba di 1 laptop Windows dan 1 laptop Linux sebelum Pertemuan 1** (cukup langkah uji §3.2/§3.3).*

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/XXXXXXXXXXXXXXXXX#offline=true&sandboxMode=true)
# 🐦🗺️ CAMAR Project 
**Proyek Penelitian S3 PSL-IPB**: Model Simulasi Prediksi Perubahan LULC (Land Use/Land Cover)

**Peneliti**: Seniarwan<sup>1</sup>, dibawah bimbingan [Prof. Baba Barus](https://scholar.google.co.id/citations?hl=id&user=zOtjie8AAAAJ)<sup>2</sup>, [Prof. Suria D. Tarigan](https://scholar.google.com/citations?user=ukdzSPsAAAAJ&hl=id&oi=ao)<sup>2</sup>, dan [Prof. Agus Buono](https://scholar.google.com/citations?user=CDIv9k0AAAAJ&hl=en)<sup>3</sup>

   <sup>1</sup> *Mahasiswa S3, Sekolah Pascasarjana, Program Studi Ilmu Pengelolaan Sumberdaya ALam dan Lingkungan, IPB University (psl24seniarwan@apps.ipb.ac.id).*
   
   <sup>2</sup> *Departemen Ilmu Tanah dan Sumberdaya Lahan, Fakultas Pertanian, IPB University, Indonesia*
   
   <sup>3</sup> *Departemen Ilmu Komputer, Sekolah Sains Data, Matematika, dan Informatika (SSMI), IPB University, Indonesia*

**CAMAR** adalah sebuah kerangka kerja pemodelan spasial yang dikembangkan dengan _Python_ untuk simulasi dan prediksi perubahan LULC di masa depan. Nama model ini dipilih sebagai akronim dari `Cellular Automata-Monte Carlo-Markov for Land Change Prediction`, sekaligus sebagai metafora atas kemampuan burung Camar dalam membaca perubahan lingkungan dan beradaptasi secara spasial.

Secara teknis, model ini mengimplementasikan metode inti `Cellular Automata-Markov (CA-Markov)` yang disempurnakan dengan simulasi `Monte Carlo` untuk memproyeksikan alokasi ruang di masa depan secara realistis.

Proyek ini dirancang untuk menjadi model fleksibel, tangguh, dan mudah diadaptasi untuk berbagai skenario penelitian, terutama dalam konteks perencanaan tata ruang, analisis dampak bencana/lingkungan, dan studi urbanisasi. Pada notebook telah tersedia data sampel yang diakses langsung melalui gdrive dan model yang dapat langsung dijalankan.

## 🧠 Metodologi

Alur kerja CAMAR dibagi menjadi dua tahap utama: proyeksi kuantitas perubahan secara non-spasial, diikuti oleh alokasi perubahan secara spasial menggunakan mesin Cellular Automata.

**Tahap 1: Proyeksi Kuantitas Perubahan (Rantai Markov)**

Tahap ini bertujuan untuk menentukan jumlah total lahan yang akan bertransisi dari satu kelas ke kelas lainnya.

- **Analisis Historis**: Model menghitung matriks transisi historis antara beberapa periode waktu (misalnya, 2015-2020 dan 2020-2024) dari data peta LULC yang ada.

- **Proyeksi Tren**: Probabilitas transisi dari matriks historis tersebut kemudian diproyeksikan ke tahun target di masa depan. CAMAR mendukung beberapa metode proyeksi, termasuk regresi linear dan kuadratik.

- **Output**: Hasil dari tahap ini adalah sebuah Matriks Transisi Proyeksi yang mendefinisikan berapa banyak sel (atau hektar) yang diharapkan akan berubah dari setiap kelas ke setiap kelas lainnya.

**Tahap 2: Alokasi Spasial (Cellular Automata-Monte Carlo)**

Setelah mengetahui jumlah perubahan, tahap ini menentukan lokasi perubahan tersebut. Ini adalah inti dari mesin simulasi CAMAR. Untuk setiap sel di dalam lanskap, probabilitas perubahan total dihitung berdasarkan empat komponen utama:

- **Matriks Transisi Proyeksi**: Memberikan probabilitas dasar perubahan untuk setiap kelas.

- **Peta Kesesuaian Lahan (Suitability Map)**: Peta eksternal yang merepresentasikan faktor pendorong spasial (misalnya, jarak ke jalan, kemiringan lereng, aksesibilitas). Peta ini memberi tahu model lokasi mana yang secara inheren lebih mungkin untuk berubah.

- **Efek Tetangga (Neighborhood Effect)**: Mensimulasikan prinsip bahwa perubahan cenderung terjadi di dekat area yang sudah berubah sebelumnya (misalnya, kota baru cenderung tumbuh di pinggiran kota yang sudah ada).

- **Seleksi "Roda Roulette" Monte Carlo**: Alih-alih selalu memilih kelas dengan probabilitas tertinggi (deterministik), CAMAR menggunakan metode Monte Carlo untuk memilih status akhir sel secara stokastik. Ini memastikan bahwa setiap kelas memiliki kemungkinan untuk dipilih secara proporsional terhadap probabilitasnya, menghasilkan pola pertumbuhan yang lebih organik dan realistis.

## 🚀 Fitur Utama

Model CAMAR dilengkapi dengan serangkaian fitur yang menjadikannya alat yang canggih dan adaptif untuk penelitian:

- **Simulasi Stokastik Modern**: Menggunakan inti CA-Monte Carlo untuk menangkap ketidakpastian dan menghasilkan pola spasial yang lebih alami, menghindari hasil yang kaku dan deterministik.

- **Performa Tinggi**: Fungsi simulasi inti ditulis dalam _Cython_, menghasilkan kecepatan yang tinggi dan memungkinkan simulasi lanskap besar dalam waktu yang wajar.

- **Kontrol Inersia**: Menyertakan parameter `change_threshold` yang memungkinkan pengguna untuk mengontrol inersia sistem, mencegah perubahan acak pada sel dengan probabilitas rendah.

- **Reproduktifitas Ilmiah**: Mendukung penggunaan `random_seed` untuk memastikan bahwa hasil simulasi stokastik dapat direproduksi sepenuhnya, yang krusial untuk penelitian dan publikasi.

- **Dua Mode Simulasi**:

    - `evolutionary`: Mode stokastik (CA-MC) untuk simulasi pertumbuhan organik.
   
    - `demand_driven`: Mode deterministik berbasis peringkat untuk alokasi berbasis target yang pasti.

- **Modular dan Dapat Diperluas**: Dibangun sebagai paket _Python_, CAMAR mudah diintegrasikan ke dalam alur kerja penelitian yang lebih besar, misalnya dengan notebook _Jupyter/Colab_.


## 🗺️ Contoh Hasil

**Log Simulasi**

```
Logging diaktifkan ke file output_data/simulation.log
Memuat data historis...
Jumlah kelas terdeteksi: 11
Tahun target yang diperbolehkan: [2030, 2035]

==================================================
TAHAP 1: VALIDASI MODEL
==================================================

--- Memvalidasi prediksi untuk 2024 dari 2020 ---
Menggunakan tren dari periode 2015-2020

--- Menjalankan Simulasi untuk Target: 2024 ---
Processing Tiles for 2024: 100%|██████████| 3234/3234 [03:20<00:00, 16.12it/s]
Merging Tiles for 2024: 100%|██████████| 3234/3234 [00:11<00:00, 284.41it/s]
Simulasi mode evolusi selesai. Hasil disimpan di: output_data/predicted_map_evolutionary_2024.tif

--- Menghitung Akurasi Model ---
Cohen's Kappa: 0.7922
Overall Jaccard Score: 0.7241
Overall Precision: 0.8436
Overall Recall: 0.8363
Overall F1-Score: 0.8362

Metrik per Kelas:
╒════════════════════════╤═════════════╤══════════╤════════════╤═══════════╕
│ Kelas                  │   Precision │   Recall │   F1-Score │   Support │
╞════════════════════════╪═════════════╪══════════╪════════════╪═══════════╡
│ Hutan                  │    0.958655 │ 0.975901 │   0.967201 │    814607 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Rawa                   │    0.623131 │ 0.838926 │   0.715103 │       745 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Mangrove               │    0.871737 │ 0.68138  │   0.764893 │     46761 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Semak Belukar          │    0.749866 │ 0.756829 │   0.753331 │    110922 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Perkebunan             │    0.899147 │ 0.926685 │   0.912709 │   2201983 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Pertanian Lahan Kering │    0.707016 │ 0.885589 │   0.786291 │   2734121 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Sawah                  │    0.900291 │ 0.834941 │   0.866385 │   3869760 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Lahan Terbangun        │    0.826431 │ 0.713616 │   0.765892 │   3376364 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Tambak                 │    0.854491 │ 0.942652 │   0.896409 │    199170 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Tubuh Air              │    0.924518 │ 0.677699 │   0.782098 │     93872 │
├────────────────────────┼─────────────┼──────────┼────────────┼───────────┤
│ Lainnya                │    0.774553 │ 0.581306 │   0.664158 │     69829 │
╘════════════════════════╧═════════════╧══════════╧════════════╧═══════════╛

==================================================
TAHAP 2: PREDIKSI LULC MASA DEPAN
==================================================

--- Memulai prediksi untuk tahun 2030 dari 2024 ---
Matriks Transisi Proyeksi untuk 2030 disimpan di output_data/projected_matrix_2030.csv

--- Menjalankan Simulasi untuk Target: 2030 ---
Processing Tiles for 2030: 100%|██████████| 3234/3234 [03:20<00:00, 16.11it/s]
Merging Tiles for 2030: 100%|██████████| 3234/3234 [00:11<00:00, 291.08it/s]Simulasi mode evolusi selesai. Hasil disimpan di: output_data/predicted_map_evolutionary_2030.tif

--- Memulai prediksi untuk tahun 2035 dari 2030 ---
Matriks Transisi Proyeksi untuk 2035 disimpan di output_data/projected_matrix_2035.csv

--- Menjalankan Simulasi untuk Target: 2035 ---

Processing Tiles for 2035: 100%|██████████| 3234/3234 [02:56<00:00, 18.29it/s]
Merging Tiles for 2035: 100%|██████████| 3234/3234 [00:11<00:00, 285.33it/s]
Simulasi mode evolusi selesai. Hasil disimpan di: output_data/predicted_map_evolutionary_2035.tif

Proses Selesai.
```
**Grafik Perubahan Luas**

![alt text](result_chart.png)

**Peta Hasil Prediksi**  

![alt text](result_map.png)



**WORK IN PROGRESS**

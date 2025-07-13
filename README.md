[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/xxxxxx#offline=true&sandboxMode=true)
# CAMAR (CA-Markov and Random Forest)
**Proyek Penelitian S3 PSL-IPB**: Model Simulasi Prediksi Perubahan LULC

**Peneliti**: Seniarwan, dibawah bimbingan [Prof. Baba Barus](https://scholar.google.co.id/citations?hl=id&user=zOtjie8AAAAJ), [Prof. Suria D. Tarigan](https://scholar.google.com/citations?user=ukdzSPsAAAAJ&hl=id&oi=ao), dan [Prof. Agus Buono](https://scholar.google.com/citations?user=CDIv9k0AAAAJ&hl=en)

**CAMAR** adalah sebuah kerangka kerja pemodelan spasial yang dikembangkan dengan Python untuk menyimulasikan dan memproyeksikan perubahan LULC di masa depan. Nama model ini dipilih sebagai akronim dari `Cellular Automata-Markov` dan `Random Forest`, sekaligus sebagai metafora atas kemampuan burung Camar dalam membaca perubahan lingkungan dan beradaptasi secara spasial.

Proyek ini dirancang untuk menjadi model fleksibel, tangguh, dan mudah diadaptasi untuk berbagai skenario penelitian, terutama dalam konteks perencanaan tata ruang, analisis dampak bencana/lingkungan, dan studi urbanisasi. Repositori ini berisi notebook eksekusi dan data sampel.

## Fitur Utama
Model CAMAR dilengkapi dengan serangkaian fitur yang menjadikannya alat yang canggih dan adaptif untuk penelitian:

- **Simulasi Dua Mode**:

   - **Mode Berbasis Tren (Evolusioner)**: Mensimulasikan perubahan lahan berdasarkan tren historis, cocok untuk skenario business-as-usual (tanpa intervensi).
   
   - **Mode Alokasi Berbasis Permintaan (Demand-Driven)**: Mensimulasikan perubahan dengan mematuhi batasan spasial (misalnya, kawasan lindung, LP2B, rencana tata ruang), ideal untuk analisis kebijakan.

- **Alur Kerja Validasi Multi-Tahap**: Sebelum melakukan prediksi, model secara otomatis menjalankan validasi jangka pendek dan/atau jangka panjang terhadap data historis untuk mengukur keandalannya menggunakan metrik standar (Cohen's Kappa, F1-Score, dll.).

- **Proyeksi Berantai Multi-Tahun**: Mampu menghasilkan prediksi untuk beberapa tahun target di masa depan secara berurutan (misalnya, 2030, 2035, 2040), di mana hasil dari satu periode menjadi input untuk periode berikutnya.

- **Konfigurasi Fleksibel**: Pengguna dapat dengan mudah menyesuaikan periode waktu historis, tahun validasi, dan tahun target prediksi hanya dengan mengubah variabel konfigurasi di notebook utama.

- **Struktur Modular**: Logika inti simulasi dipisahkan ke dalam modul pustaka `camar`, sementara notebook `CAMAR_simulation.ipynb` berfungsi sebagai antarmuka yang mudah digunakan.

- **Komputasi Paralel Otomatis (Multiprocessing)**: Untuk efisiensi pada data raster besar, CAMAR secara otomatis membagi raster menjadi tile kecil (misal 128×128 piksel). Setiap tile diproses paralel menggunakan seluruh core CPU, kemudian hasilnya digabungkan kembali secara seamless. Mekanisme padding diterapkan untuk menghindari artefak di batas tile, sehingga hasil tetap konsisten secara spasial.

## Metodologi

1. **Penyusunan Data Historis**  
   Kumpulkan peta LULC multi-temporal (misal: tahun `t₀, t₁, t₂`) dalam bentuk raster/grid.  
   Siapkan data prediktor spasial (variabel geobiofisik-lingkungan, sosial ekonomi, dll) untuk *suitability map*.
   
2. **Penyusunan Suitability/Probability Map dengan Random Forest**  
   Suitability/probability map adalah raster yang menunjukkan kecocokan setiap piksel untuk tiap kelas.  
   Model Random Forest digunakan untuk memetakan hubungan antara piksel (dengan variabel prediktor $X$) dan kelas lahan target:

   $S_{i, c} = RF_c(X_i)$
   
   - $S_{i, c}$ = Skor suitability piksel $i$ untuk kelas $c$  
   - $RF_c$ = Model Random Forest kelas $c$  
   - $X_i$ = Vektor fitur/prediktor di piksel $i$
   
   Hasil prediksi $S$ ditunjukkan dengan nilai probabilitas [0, 1] per kelas LULC.   

3. **Perhitungan Matriks Transisi Markov**  
   Hitung matriks transisi probabilitas antar kelas lahan berdasarkan dua waktu historis.  
      
   $$P_{i,j} = \frac{N_{i \rightarrow j}}{\sum_{k} N_{i \rightarrow k}}$$
   
   - $P_{i,j}$ = Probabilitas perubahan dari kelas $i$ ke kelas $j$  
   - $N_{i \rightarrow j}$ = Jumlah piksel berpindah dari $i$ ke $j$

4. **Interpolasi Matriks Transisi**  
   Untuk prediksi masa depan, matriks transisi diinterpolasi linier berdasarkan dua periode historis.
     
   $$\mathbf{P}_{\mathrm{proj}} = \mathbf{P}_B + \frac{(\mathbf{P}_B - \mathbf{P}_A)}{\Delta t{A \rightarrow B}} \cdot (t{\mathrm{target}} - t_B)$$
   
   - $\mathbf{P}_A$, $\mathbf{P}_B$ = Matriks transisi dari dua periode historis  
   - $t_{target}$ = Tahun prediksi  
   - $t_B$ = Tahun akhir data historis

5. **Perhitungan Efek Spasial (Neighborhood/Contiguity)**  
   Menggunakan kernel Moore 5x5:

   ```
   0  0  1  0  0 
   0  1  1  1  0
   1  1  1  1  1
   0  1  1  1  0
   0  0  1  0  0
   ```


   Efek kontiguitas dihitung melalui konvolusi:
   
   $C_{i,c} = \text{Convolve2D}(\mathbb{I}(y = c), K_{5x5}) + \delta$
   
   Di mana $\delta$ adalah offset kecil untuk menghindari nol.

6. **Simulasi Cellular Automata (CA)**  
   Untuk setiap piksel, peluang transisi kelas dihitung berdasarkan:
   
   $P'{i,j} = P{i,j} \cdot S_{i,j} \cdot C_{i,j}$

   Probabilitas dinormalisasi dan proses update dilakukan untuk setiap waktu prediksi.

7. **Validasi dan Visualisasi**  
   Hasil prediksi diverifikasi menggunakan metrik spasial: Cohen’s Kappa, Jaccard, F1-Score, dan analisis perubahan spasial.

 **WORK IN PROGRESS**

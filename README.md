# 🍎 Deteksi Kualitas apel — HSV Color Segmentation

> Sistem deteksi kualitas visual buah apel berbasis **Python & OpenCV** menggunakan segmentasi ruang warna HSV, dikembangkan sebagai bagian dari **Ujian Tengah Semester (UTS) Matakuliah Pengolahan Citra Digital**.  
> Program Studi Teknologi Rekayasa Perangkat Lunak — Politeknik Negeri Madiun

---

## 📋 Deskripsi Project

Project ini mengimplementasikan dan membandingkan dua pipeline pemrosesan citra untuk mengklasifikasikan kelayakan konsumsi buah apel secara otomatis berdasarkan analisis warna kulit.

| Pipeline | Deskripsi |
|----------|-----------|
| **Pipeline Jurnal** | Implementasi ulang metode Suradi dkk. (2023) — klasifikasi kematangan (Belum Matang / Setengah Matang / Matang) |
| **Pipeline Modifikasi** | Pengembangan mandiri — menambahkan Brown Masking, standarisasi ukuran, dan Stratified Split 80:20 |

Tujuan utama project ini bukan sekadar mendeteksi kematangan, tetapi menentukan **kelayakan konsumsi** demi mendukung pengurangan *food waste* selaras dengan **SDG 2: Zero Hunger**.

---

## ✨ Fitur Utama

- **Segmentasi 4 Kanal Warna** — Hijau, Kuning, Merah, dan Cokelat (busuk)
- **Brown Masking** — deteksi area pembusukan (*bruising*) yang tidak tertangkap pipeline standar
- **Klasifikasi Prioritas Keamanan Pangan** — busuk terdeteksi lebih dahulu sebelum mengecek kematangan
- **Standarisasi Ukuran Citra** — semua gambar di-resize ke 400×400 px untuk konsistensi
- **Stratified Split 80:20** — evaluasi pada *unseen data* agar hasil akurasi objektif
- **Visualisasi Mask** — output gambar 5-panel (Original + 4 mask warna) untuk setiap citra

---

## 🗂️ Struktur Dataset

Dataset disimpan di Google Drive dengan struktur folder sebagai berikut:

```
Apple/
├── Fresh/          # Apel segar → label: Matang (Layak Konsumsi)
│   ├── apple1.jpg
│   ├── apple2.jpg
│   └── ...
└── Rotten/         # Apel busuk → label: Rotten (Tidak Layak Konsumsi)
    ├── apple1.jpg
    ├── apple2.jpg
    └── ...
```

---

## 🔧 Teknologi & Library

| Library | Versi | Kegunaan |
|---------|-------|----------|
| `opencv-python` | ≥ 4.x | Pemrosesan citra, konversi warna, masking |
| `numpy` | ≥ 1.x | Operasi array & penghitungan piksel |
| `matplotlib` | ≥ 3.x | Visualisasi mask dan output |
| `scikit-learn` | ≥ 1.x | `train_test_split` untuk Stratified Split |
| Google Colab | — | Runtime & akses Google Drive |

---

## 🚀 Cara Menjalankan

### 1. Buka di Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1RWZA-QK7c6_Gvtn-3KQZKNKElZ3suMm3).

### 2. Mount Google Drive

```python
from google.colab import drive
drive.mount('/content/drive')
```

### 3. Pastikan struktur folder dataset sudah sesuai

```
/content/drive/MyDrive/Apple/Fresh/
/content/drive/MyDrive/Apple/Rotten/
```

### 4. Install dependensi (jika diperlukan)

```bash
pip install opencv-python numpy matplotlib scikit-learn
```

### 5. Jalankan Pipeline

**Pipeline Jurnal (Baseline):**
```python
dataset_root = '/content/drive/MyDrive/Apple/'
proses_dataset(dataset_root)
```

**Pipeline Modifikasi:**
```python
dataset_root = '/content/drive/MyDrive/Apple'
run_pipeline(dataset_root)
```

**Perbandingan Kedua Pipeline:**
```python
dataset_root = '/content/drive/MyDrive/Apple'
bandingkan_pipeline(dataset_root)
```

---

## ⚙️ Detail Teknis Pipeline Modifikasi

### Tahap 1 — Preprocessing

```python
img_resized = cv2.resize(img, (400, 400))
hsv_img = cv2.cvtColor(img_resized, cv2.COLOR_BGR2HSV)
```

### Tahap 2 — Segmentasi Warna (4 Mask)

| Mask | Rentang HSV Bawah | Rentang HSV Atas | Indikasi |
|------|-------------------|------------------|----------|
| Hijau | `[35, 40, 40]` | `[85, 255, 255]` | Belum matang |
| Kuning | `[20, 40, 40]` | `[34, 255, 255]` | Setengah matang |
| Merah | `[0, 40, 40]` + `[160, 40, 40]` | `[10, 255, 255]` + `[180, 255, 255]` | Matang |
| **Cokelat** | `[0, 0, 0]` | `[30, 255, 100]` | **Busuk / kerusakan** |

### Tahap 3 — Klasifikasi (Logika Prioritas)

```python
if fitur['per_b'] > 0.12:       # Prioritas 1: Keamanan pangan
    prediksi = "Rotten"
elif fitur['per_g'] > 0.5:      # Prioritas 2: Belum matang
    prediksi = "Belum Matang"
elif fitur['per_y'] > 0.6:      # Prioritas 3: Setengah matang
    prediksi = "Setengah Matang"
else:                            # Default: Matang
    prediksi = "Matang"
```

### Tahap 4 — Kelayakan Konsumsi

| Prediksi | Status Kelayakan |
|----------|-----------------|
| Matang | ✅ Layak Konsumsi |
| Rotten | ❌ Tidak Layak Konsumsi |
| Belum Matang / Setengah Matang | ⚠️ Perlu Pemeriksaan |

---

## 📊 Perbedaan Pipeline Jurnal vs Modifikasi

| Aspek | Pipeline Jurnal | Pipeline Modifikasi |
|-------|----------------|---------------------|
| Input | Ukuran dinamis | Terstandarisasi 400×400 px |
| Jumlah mask | 3 kanal | 4 kanal (+ Brown Mask) |
| Deteksi busuk | ❌ Tidak ada | ✅ Brown Masking |
| Validasi | Seluruh dataset | Stratified test set 20% |
| Output kelayakan | Label kematangan saja | Layak / Tidak Layak / Perlu Pemeriksaan |

---

## 📁 Struktur File

```
.
├── UTSPengolahanCitra.ipynb    # Notebook utama (semua pipeline)
└── README.md                   # Dokumentasi project ini
```

---

## 📚 Referensi

1. Gonzalez, R. C., & Woods, R. E. (2018). *Digital Image Processing* (4th ed., Global Edition). Pearson.  
2. MahersatillahSuradi, A. A., Rasyid, M. F., & Rizal, M. (2023). Deteksi Tingkat Kematangan Buah Apel Menggunakan Segmentasi Ruang Warna HSV.
3. Rizka Rizka, S. Nasution, F. Aulia, dan Supiyandi Supiyandi, “Penerapan Metode Segmentasi Warna HSV untuk Deteksi Objek Berbasis Warna pada Citra Digital,” Router, vol. 3, no. 4, hlm. 11–24, Des 2025, doi: 10.62951/router.v3i4.706
---

## 👤 Author

**Mohammad Fakhriza M**
**Muhammad Aulia Al Farouq**
NIM: 234311018  
NIM: 234311020
Program Studi Teknologi Rekayasa Perangkat Lunak  
Politeknik Negeri Madiun  

---

> *Project ini dikembangkan sebagai tugas UTS Matakuliah Pengolahan Citra Digital dan bertujuan untuk berkontribusi pada upaya pengurangan food waste melalui otomatisasi sistem sortir buah berbasis visi komputer.*

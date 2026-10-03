# Dokumen Desain Awal P2

## 1. Judul

Klasifikasi Tanaman Durian dan Papaya pada Citra UAV Menggunakan Transfer Learning.

## 2. Latar Belakang

Citra UAV dapat digunakan untuk membantu identifikasi tanaman pada area pertanian. Project ini membangun model klasifikasi untuk membedakan tanaman durian dan papaya menggunakan citra UAV dari dataset InterDuPa-UAV.

Model yang digunakan adalah ResNet18 karena arsitekturnya cukup ringan untuk dibandingkan dalam beberapa skenario pelatihan dan dapat memanfaatkan bobot pretrained ImageNet.

## 3. Tujuan

1. Membangun model klasifikasi dua kelas tanaman dari citra UAV.
2. Membandingkan Feature Extraction, Partial Fine-Tuning, dan Training from Scratch.
3. Mengukur akurasi, waktu pelatihan, dan latensi inference.
4. Menentukan konfigurasi model yang paling sesuai berdasarkan hasil evaluasi.

## 4. Dataset

Dataset yang digunakan adalah **InterDuPa-UAV: A UAV-based Dataset for the Classification of Intercropped Durian and Papaya Trees**, yang diperoleh dari Zenodo:

<https://zenodo.org/records/15664908>

Dataset lokal memiliki dua kelas:

| Kelas | Jumlah citra |
| --- | ---: |
| Durian (`Durio zibethinus`) | 3.327 |
| Papaya (`Carica papaya`) | 2.872 |

Dataset mentah tidak disimpan di GitHub karena ukuran file citra besar. Sumber dan instruksi penyiapannya dijelaskan di [`datasets/README.md`](datasets/README.md).

## 5. Rancangan Sistem

Alur sistem yang dirancang:

```text
Citra UAV
   ↓
Pemeriksaan dataset dan label
   ↓
Pembagian train-validation 80:20
   ↓
Preprocessing dan augmentasi citra
   ↓
ResNet18 dengan salah satu dari tiga strategi pelatihan
   ↓
Prediksi kelas Durian atau Papaya
   ↓
Evaluasi akurasi, confusion matrix, waktu training, dan latency
```

Preprocessing menggunakan ukuran input 224 × 224 piksel, konversi RGB, normalisasi ImageNet, serta augmentasi pada data training berupa random crop, horizontal flip, dan color jitter. Data validation tidak diberi augmentasi acak.

## 6. Skenario Eksperimen

| Mode | Rancangan |
| --- | --- |
| Feature Extraction | ResNet18 pretrained; seluruh backbone dibekukan dan hanya classifier yang dilatih. |
| Partial Fine-Tuning | ResNet18 pretrained; layer akhir dan classifier dilatih, layer awal dibekukan. |
| Training from Scratch | ResNet18 tanpa bobot pretrained; seluruh parameter dilatih dari inisialisasi acak. |

Ketiga mode menggunakan split dataset, preprocessing, jumlah epoch, dan batch size yang sama agar hasil dapat dibandingkan secara adil.

## 7. Evaluasi

Evaluasi dilakukan menggunakan:

- train loss dan train accuracy;
- validation loss dan validation accuracy;
- best validation accuracy dan epoch terbaik;
- confusion matrix;
- waktu training setiap mode; dan
- inference latency model terbaik dalam milidetik per citra.

Hasil disimpan di direktori `results/` dalam bentuk tabel CSV, grafik akurasi, confusion matrix, dan konfigurasi eksperimen.

## 8. Batasan dan Risiko

- Dataset mentah tidak disimpan di repository karena ukurannya besar.
- Metadata yang tersedia bersifat deskriptif dan belum menyediakan pemetaan setiap crop ke citra UAV sumber.
- Oleh karena itu, split stratified berbasis citra belum dapat menjamin bebas data leakage antar-citra sumber UAV.
- Hasil eksperimen bergantung pada perangkat dan environment saat training, terutama GPU yang digunakan.

## 9. Keluaran yang Direncanakan

1. Notebook eksperimen yang dapat dijalankan ulang.
2. Tabel perbandingan tiga mode pelatihan.
3. Grafik akurasi per epoch.
4. Confusion matrix.
5. Latensi inference model terpilih.
6. README ringkas untuk menjelaskan hasil dan kesimpulan.

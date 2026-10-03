# P2 — Klasifikasi Tanaman pada Citra UAV

Project ini membandingkan tiga strategi pelatihan ResNet18 untuk klasifikasi dua kelas tanaman dari citra UAV: **Durian** (*Durio zibethinus*) dan **Papaya** (*Carica papaya*).

## Dataset

Dataset yang digunakan adalah [InterDuPa-UAV](https://zenodo.org/records/15664908), dengan total 6.199 citra:

| Kelas | Jumlah citra |
|---|---:|
| Durian | 3.327 |
| Papaya | 2.872 |
| **Total** | **6.199** |

Dataset mentah dan `Dataset-metadata.xlsx` tidak disimpan di repository karena ukuran file dan keterbatasan struktur metadata. Repository menyertakan [`datasets/metadata.csv`](datasets/metadata.csv), yaitu manifest 6.199 citra dan label yang dibuat dari folder kelas. Manifest ini bukan pengganti metadata sumber UAV dan belum memiliki `group_id`. Cara memperoleh dataset dan menjalankan notebook dijelaskan di [`datasets/README.md`](datasets/README.md).

## Metode

Notebook [`notebook-p2-vision-dan-deep.ipynb`](notebook-p2-vision-dan-deep.ipynb) adalah notebook eksperimen yang dijalankan pada Kaggle GPU menggunakan dataset Kaggle. Notebook yang sama dapat dijalankan di luar Kaggle setelah dataset disiapkan di `datasets/`. Notebook menggunakan input 224×224 piksel, normalisasi ImageNet, augmentasi pada data training, seed 42, batch size 32, dan 10 epoch. Sebanyak 20 citra dari setiap kelas ditahan sebagai blind test sebelum train-validation split. Ketiga mode menggunakan sisa data, split, dan preprocessing yang sama:

1. **Feature Extraction** — backbone ResNet18 pretrained dibekukan; hanya classifier yang dilatih.
2. **Partial Fine-Tuning** — layer akhir dan classifier dilatih, sedangkan layer awal dibekukan.
3. **Training from Scratch** — ResNet18 dilatih tanpa bobot pretrained.

## Hasil eksperimen

| Mode | Validation accuracy terbaik | Epoch terbaik | Waktu training |
|---|---:|---:|---:|
| Partial Fine-Tuning | **100,00%** | 1 | 788,66 detik |
| Scratch | **100,00%** | 7 | 857,65 detik |
| Feature Extraction | 99,68% | 4 | 771,50 detik |

Model terpilih adalah **Partial Fine-Tuning** karena memperoleh validation accuracy tertinggi bersama Scratch dan mencapai akurasi tersebut lebih cepat, yaitu pada epoch 1. Feature Extraction memiliki waktu training terendah, tetapi validation accuracy-nya sedikit lebih rendah. Latensi inference Partial Fine-Tuning adalah **3,157 ms per citra** pada GPU environment eksperimen.

Hasil lengkap tersedia di:

- [`results/comparison.csv`](results/comparison.csv) — tabel perbandingan dan latency.
- [`results/accuracy_curve.png`](results/accuracy_curve.png) — grafik akurasi/loss per epoch.
- [`results/confusion_matrix.png`](results/confusion_matrix.png) — confusion matrix model terpilih.
- [`results/blind_test_images.png`](results/blind_test_images.png) — visualisasi 40 citra blind test anonim beserta prediksi model.
- [`results/config.json`](results/config.json) — konfigurasi eksperimen.
- [`results/best_resnet18.pt`](results/best_resnet18.pt) — bobot model Partial Fine-Tuning terpilih.
- [`datasets/metadata.csv`](datasets/metadata.csv) — manifest path, label numerik, dan nama kelas.

## Analisis dan keterbatasan

Transfer learning memberikan hasil sangat baik pada dataset ini. Partial Fine-Tuning dan Scratch sama-sama mencapai validation accuracy 100%, tetapi Partial Fine-Tuning mencapainya pada epoch pertama dan membutuhkan waktu training lebih singkat daripada Scratch. Blind test yang terdiri dari 40 citra anonim juga menghasilkan akurasi internal 100%. Hasil tersebut perlu dibaca dengan hati-hati karena blind test dan validation split masih dilakukan pada level citra. Metadata belum menyediakan `group_id` sumber UAV, sehingga codebase belum dapat memastikan tidak terjadi data leakage antar-crop dari sumber citra yang sama.

Jumlah kelas pada folder citra yang digunakan notebook adalah Durian 3.327 dan Papaya 2.872. Teks deskriptif di `Dataset-metadata.xlsx` mencantumkan pasangan jumlah yang terbalik, sehingga jumlah pada folder citra digunakan sebagai acuan eksperimen dan perbedaan ini perlu diverifikasi terhadap sumber dataset.

## Dokumen

- [`DESAIN_AWAL.md`](DESAIN_AWAL.md) — desain awal proyek.
- [`datasets/README.md`](datasets/README.md) — sumber dan cara menyiapkan dataset.

# P2 — Klasifikasi Tanaman pada Citra UAV

Project ini membandingkan tiga strategi pelatihan ResNet18 untuk klasifikasi dua kelas tanaman dari citra UAV: **Durian** (*Durio zibethinus*) dan **Papaya** (*Carica papaya*).

## Dataset

Dataset yang digunakan adalah [InterDuPa-UAV](https://zenodo.org/records/15664908), dengan total 6.199 citra:

| Kelas | Jumlah citra |
|---|---:|
| Durian | 3.327 |
| Papaya | 2.872 |
| **Total** | **6.199** |

Dataset mentah tidak disimpan di repository karena ukuran file citra besar. Cara menyiapkannya dijelaskan di [`datasets/README.md`](datasets/README.md). Metadata sumber yang tersedia berupa `Dataset-metadata.xlsx` dan bersifat deskriptif; belum ada pemetaan setiap crop ke citra UAV sumber.

## Metode

Notebook [`notebook.ipynb`](notebook.ipynb) menggunakan input 224×224 piksel, normalisasi ImageNet, augmentasi pada data training, seed 42, batch size 32, dan 10 epoch. Ketiga mode menggunakan split dan preprocessing yang sama:

1. **Feature Extraction** — backbone ResNet18 pretrained dibekukan; hanya classifier yang dilatih.
2. **Partial Fine-Tuning** — layer akhir dan classifier dilatih, sedangkan layer awal dibekukan.
3. **Training from Scratch** — ResNet18 dilatih tanpa bobot pretrained.

## Hasil eksperimen

| Mode | Validation accuracy terbaik | Epoch terbaik | Waktu training |
|---|---:|---:|---:|
| Partial Fine-Tuning | **100,00%** | 3 | 408,51 detik |
| Scratch | 99,60% | 10 | 434,25 detik |
| Feature Extraction | 99,19% | 2 | 422,03 detik |

Model terpilih adalah **Partial Fine-Tuning** karena memperoleh validation accuracy tertinggi dan waktu training terendah pada eksperimen ini. Latensi inference model tersebut adalah **3,07 ms per citra** pada GPU environment eksperimen.

Hasil lengkap tersedia di:

- [`results/comparison.csv`](results/comparison.csv) — tabel perbandingan dan latency.
- [`results/accuracy_curve.png`](results/accuracy_curve.png) — grafik akurasi/loss per epoch.
- [`results/confusion_matrix.png`](results/confusion_matrix.png) — confusion matrix model terpilih.
- [`results/config.json`](results/config.json) — konfigurasi eksperimen.

## Analisis dan keterbatasan

Transfer learning memberikan hasil sangat baik pada dataset ini. Partial Fine-Tuning sedikit mengungguli Feature Extraction dan Scratch, sehingga penyesuaian layer akhir ResNet18 membantu model beradaptasi terhadap citra tanaman UAV. Namun, hasil ini perlu dibaca dengan hati-hati karena split yang digunakan adalah stratified split berbasis citra. Metadata yang tersedia belum menyediakan `group_id` sumber UAV, sehingga codebase belum dapat memastikan tidak terjadi data leakage antar-crop dari sumber citra yang sama.

## Dokumen

- [`DESAIN_AWAL.md`](DESAIN_AWAL.md) — desain awal proyek.
- [`DESAIN_LENGKAP.md`](DESAIN_LENGKAP.md) — rancangan dan penjelasan lengkap.

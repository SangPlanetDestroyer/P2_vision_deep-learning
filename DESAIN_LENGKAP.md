# P2 — Transfer Learning untuk Klasifikasi Tanaman pada Citra UAV

## 1. Konteks Proyek

Project ini merupakan implementasi tugas **P2 Computer Vision dan Deep Learning** yang berfokus pada penerapan **Transfer Learning** untuk klasifikasi citra tanaman yang diperoleh menggunakan **UAV (Unmanned Aerial Vehicle)**.

Dataset yang digunakan adalah **InterDuPa-UAV**, yaitu dataset citra UAV untuk klasifikasi tanaman **Durian (_Durio zibethinus_)** dan **Papaya (_Carica papaya_)** pada lingkungan pertanian.

Model utama yang digunakan adalah **ResNet18 pretrained pada ImageNet**. Eksperimen dilakukan untuk membandingkan tiga pendekatan pembelajaran:

1. **Feature Extraction**
2. **Partial Fine-Tuning**
3. **Training from Scratch**

Perbandingan ketiga pendekatan tersebut digunakan untuk mengetahui pengaruh pemanfaatan fitur pretrained terhadap performa klasifikasi pada dataset tanaman dari citra UAV.

---

## 2. Tujuan

Tujuan utama project ini adalah:

- Membangun model klasifikasi tanaman berbasis citra UAV.
- Menerapkan Transfer Learning menggunakan ResNet18 pretrained ImageNet.
- Membandingkan Feature Extraction, Partial Fine-Tuning, dan Training from Scratch.
- Mengevaluasi performa model menggunakan validation accuracy dan metrik evaluasi lainnya.
- Mengukur waktu training dan latency inference model.
- Menganalisis pengaruh strategi Transfer Learning terhadap performa model.
- Menyusun pipeline eksperimen yang reproducible dan dapat dijalankan menggunakan computational resource dari Kaggle.

---

## 3. Dataset

Dataset utama yang digunakan adalah **InterDuPa-UAV**.

Dataset terdiri dari citra UAV yang digunakan untuk membedakan dua kelas tanaman:

| Class  | Scientific Name    |
| ------ | ------------------ |
| Durian | _Durio zibethinus_ |
| Papaya | _Carica papaya_    |

Dataset berasal dari citra UAV dan menyediakan data tanaman yang telah diekstraksi dari citra udara.

Struktur dataset lokal:

```text
datasets/
├── 25 meter/
├── 30 meter/
├── Dataset-metadata.xlsx
├── metadata.csv
├── demo.py
├── Durian (durio zibethinus)/
├── Papaya (carica papaya)/
└── rar/
```

`Dataset-metadata.xlsx` digunakan sebagai sumber informasi deskriptif dataset.
`datasets/metadata.csv` adalah manifest repository yang berisi
`relative_path`, `label`, dan `class_name` untuk setiap citra kelas. Manifest
ini tidak menyediakan pemetaan crop ke citra UAV sumber.

### Sumber Dataset

InterDuPa-UAV:

https://zenodo.org/records/15664908

> Dataset mentah dan `Dataset-metadata.xlsx` tidak disimpan di repository GitHub karena ukuran file dan keterbatasan struktur metadata. Repository tetap menyimpan `datasets/metadata.csv` sebagai manifest path dan label, serta dokumentasi cara memperoleh dataset.

---

## 4. Permasalahan Data Leakage

Dataset tanaman berasal dari citra UAV yang kemudian menghasilkan sejumlah crop/image individual. Oleh karena itu, beberapa gambar dapat berasal dari **satu citra UAV atau satu kondisi akuisisi yang sama**.

Pembagian dataset secara acak pada level gambar berpotensi menyebabkan **data leakage**, karena gambar yang sangat mirip atau berasal dari sumber citra UAV yang sama dapat masuk ke train dan validation.

Oleh karena itu, proses train-validation split harus mempertimbangkan informasi pada:

```text
Dataset-metadata.xlsx
```

Jika metadata menyediakan identifier sumber citra, sesi penerbangan, lokasi, atau informasi akuisisi lainnya, pembagian data harus dilakukan berdasarkan identifier tersebut, bukan sekadar secara acak pada setiap gambar.

Prinsip yang digunakan:

```text
Raw UAV Images
       │
       ▼
Metadata / Source Group
       │
       ├───────────────┐
       ▼               ▼
     Train           Validation
```

Tujuannya adalah membuat validation set lebih representatif terhadap kemampuan generalisasi model pada data UAV yang belum pernah dilihat sebelumnya.

---

## 5. Pendekatan Transfer Learning

Model utama yang digunakan adalah:

```text
ResNet18 + ImageNet pretrained weights
```

ResNet18 dipilih karena merupakan model yang relatif ringan dan sesuai dengan eksperimen Transfer Learning pada materi P2.

Input model menggunakan preprocessing yang sesuai dengan pretrained ImageNet:

```text
Resize / Crop → RGB → Tensor → Normalize
```

Normalisasi:

```python
mean = [0.485, 0.456, 0.406]
std  = [0.229, 0.224, 0.225]
```

Ukuran input:

```text
224 × 224 pixels
```

---

## 6. Eksperimen Model

Tiga pendekatan akan dibandingkan.

### 6.1 Feature Extraction

Backbone ResNet18 pretrained dibekukan.

Hanya classifier/head yang dilatih pada dataset tanaman.

```text
Image
  │
  ▼
ResNet18 pretrained
(frozen)
  │
  ▼
Classification Head
(trainable)
  │
  ▼
Durian / Papaya
```

Karakteristik:

- Backbone menggunakan fitur dari ImageNet.
- Parameter backbone tidak diperbarui.
- Hanya fully connected layer/classification head yang dilatih.
- Learning rate dapat menggunakan sekitar `1e-3`.

---

### 6.2 Partial Fine-Tuning

Sebagian backbone ResNet18 dibekukan, sedangkan bagian akhir model dan classification head dilatih.

Konfigurasi utama:

```text
layer1 ── frozen
layer2 ── frozen
layer3 ── frozen
layer4 ── trainable
fc     ── trainable
```

Tujuannya adalah mempertahankan fitur umum dari ImageNet sekaligus memungkinkan model menyesuaikan fitur tingkat tinggi terhadap karakteristik citra tanaman UAV.

Learning rate yang digunakan dapat dibedakan antara backbone dan classification head.

Contoh:

```text
layer4 : 1e-4
fc     : 1e-3
```

---

### 6.3 Training from Scratch

ResNet18 digunakan tanpa pretrained ImageNet weights.

Semua parameter model diinisialisasi secara acak dan dilatih menggunakan dataset tanaman.

```text
Image
  │
  ▼
ResNet18
(random initialization)
  │
  ▼
Classification Head
  │
  ▼
Durian / Papaya
```

Pendekatan ini digunakan sebagai baseline untuk mengetahui seberapa besar keuntungan yang diberikan oleh Transfer Learning.

---

## 7. Perbandingan Eksperimen

Eksperimen utama:

| Experiment          | Backbone | Trainable Layers | Pretrained | Learning Rate |
| ------------------- | -------- | ---------------- | ---------- | ------------- |
| Feature Extraction  | ResNet18 | FC               | ImageNet   | `1e-3`        |
| Partial Fine-Tuning | ResNet18 | Layer4 + FC      | ImageNet   | `1e-4 / 1e-3` |
| Scratch             | ResNet18 | All layers       | No         | `1e-3`        |

Setiap eksperimen akan menggunakan konfigurasi dataset dan preprocessing yang konsisten agar perbandingan lebih valid.

---

## 8. Data Augmentation

Training data dapat menggunakan augmentation untuk meningkatkan variasi data dan membantu generalisasi model.

Augmentasi yang digunakan mengikuti pendekatan pada materi P2, antara lain:

```text
RandomResizedCrop
HorizontalFlip
ColorJitter
```

Validation data tidak menggunakan augmentation yang mengubah karakteristik gambar secara acak.

Pipeline secara umum:

```text
Training:
Image
  │
  ├── RandomResizedCrop
  ├── HorizontalFlip
  ├── ColorJitter
  ├── Resize / Crop
  ├── ToTensor
  └── ImageNet Normalize

Validation:
Image
  │
  ├── Resize / Crop
  ├── ToTensor
  └── ImageNet Normalize
```

---

## 9. Training

Training dilakukan menggunakan PyTorch.

Parameter eksperimen seperti:

- batch size
- learning rate
- optimizer
- jumlah epoch
- scheduler
- random seed
- model configuration

akan dicatat agar eksperimen dapat direproduksi.

Scheduler yang digunakan dapat mengikuti materi P2:

```text
CosineAnnealingLR
```

Jumlah epoch awal mengikuti rancangan eksperimen pada materi:

```text
10 epochs
```

Konfigurasi dapat disesuaikan apabila eksperimen menunjukkan kebutuhan untuk evaluasi tambahan.

---

## 10. Evaluasi

Model akan dievaluasi berdasarkan beberapa aspek.

### Performance

Minimal:

```text
Train Accuracy
Train Loss
Validation Loss
Validation Accuracy
```

Selain itu dapat digunakan:

```text
Precision
Recall
F1-score
Confusion Matrix
```

untuk melihat performa klasifikasi masing-masing kelas.

### Training Performance

Dicatat:

```text
Training Time
Best Validation Accuracy
Epoch dengan Best Validation Accuracy
```

### Inference Performance

Untuk model terpilih akan diukur:

```text
Inference Latency
```

sebagai salah satu pertimbangan penerapan model pada sistem computer vision berbasis robot/UAV.

---

## 11. Output Eksperimen

Hasil eksperimen akan disimpan dalam bentuk:

```text
results/
├── comparison.csv
├── accuracy_curve.png
├── confusion_matrix.png
└── ...
```

`comparison.csv` digunakan untuk membandingkan hasil tiga pendekatan:

```text
Feature Extraction
Partial Fine-Tuning
Scratch
```

Contoh informasi yang dicatat:

```text
model
best_val_accuracy
best_epoch
training_time
inference_latency
```

---

## 12. Environment

Eksperimen utama dijalankan menggunakan **Kaggle Notebook** untuk memanfaatkan computational resource/GPU.

Dataset pada Kaggle akan tersedia melalui:

```text
/kaggle/input/
```

Sedangkan hasil eksperimen sementara dapat disimpan pada:

```text
/kaggle/working/
```

Struktur umum:

```text
/kaggle/input/
└── interdupa-uav/

/kaggle/working/
├── checkpoints/
├── results/
└── logs/
```

Notebook harus dapat dijalankan dari awal sampai akhir tanpa bergantung pada file yang hanya tersedia di komputer lokal.

---

## 13. Struktur Codebase

Struktur repository saat ini:

```text
P2/
├── datasets/
│   ├── 25 meter/
│   ├── 30 meter/
│   ├── Dataset-metadata.xlsx
│   ├── metadata.csv
│   ├── demo.py
│   ├── Durian (durio zibethinus)/
│   ├── Papaya (carica papaya)/
│   └── rar/
│
├── materi_p2/
│   ├── IJCCS_FruitAndVeg_resnet18.ipynb
│   └── RET503_Pertemuan3_Transfer_Learning.pdf
│
├── notebook-p2-vision-dan-deep.ipynb
└── README.md
```

Keterangan:

- `datasets/` — dataset dan metadata lokal.
- `materi_p2/` — materi pembelajaran dan contoh implementasi dari pertemuan P2.
- `notebook-p2-vision-dan-deep.ipynb` — notebook utama eksperimen yang dijalankan pada Kaggle GPU dan menyimpan output eksperimen.
- `README.md` — dokumentasi project.

---

## 14. Rencana Pipeline Notebook

Notebook utama dirancang dengan tahapan:

```text
01. Configuration
        ↓
02. Dataset Inspection
        ↓
03. Metadata Inspection
        ↓
04. Blind Test Holdout (20 citra per kelas)
        ↓
05. Data Leakage Analysis
        ↓
06. Train / Validation Split
        ↓
07. Dataset Visualization
        ↓
08. Data Augmentation & Preprocessing
        ↓
09. ResNet18 Preparation
        ↓
10. Feature Extraction
        ↓
11. Partial Fine-Tuning
        ↓
12. Training from Scratch
        ↓
13. Model Comparison
        ↓
14. Accuracy Curves
        ↓
15. Confusion Matrix
        ↓
16. Inference Latency
        ↓
17. Blind Test Prediction
        ↓
18. Final Analysis
```

---

## 15. Prinsip Reproducibility

Agar eksperimen dapat direproduksi:

- Gunakan random seed yang tetap.
- Catat konfigurasi setiap eksperimen.
- Gunakan preprocessing yang konsisten.
- Gunakan split dataset yang sama untuk ketiga pendekatan.
- Jangan melakukan split ulang secara berbeda untuk setiap model.
- Simpan hasil training dan validation.
- Simpan konfigurasi hyperparameter.
- Dokumentasikan environment yang digunakan.

Perbandingan model harus dilakukan pada kondisi dataset dan validation set yang sama.

---

## 16. Target Akhir

Output akhir project diharapkan berupa:

1. Dataset InterDuPa-UAV yang terstruktur.
2. Metadata dataset yang telah dianalisis.
3. Strategi train-validation split yang reproducible dengan keterbatasan data leakage yang didokumentasikan.
4. Implementasi ResNet18 Feature Extraction.
5. Implementasi ResNet18 Partial Fine-Tuning.
6. Implementasi ResNet18 Training from Scratch.
7. Perbandingan performa ketiga pendekatan.
8. Grafik accuracy per epoch.
9. Confusion matrix.
10. Pengukuran training time.
11. Pengukuran inference latency.
12. Analisis hasil eksperimen.
13. Blind test anonim sebanyak 20 citra per kelas.
14. Notebook yang dapat direproduksi pada Kaggle.
15. Dokumentasi project pada GitHub.

---

## 17. Status Pengembangan

Status implementasi saat ini:

```text
[x] Dataset diperoleh
[x] Struktur project dibuat
[x] Materi Transfer Learning dipelajari
[x] Analisis Dataset-metadata.xlsx
[x] Analisis potensi data leakage
[x] Implementasi train-validation split yang reproducible
[ ] Menentukan split final bebas leakage berbasis source group (group mapping belum tersedia)
[x] Implementasi preprocessing dan augmentasi
[x] Implementasi Feature Extraction
[x] Implementasi Partial Fine-Tuning
[x] Implementasi Training from Scratch
[x] Training pada Kaggle GPU
[x] Perbandingan hasil
[x] Evaluasi latency
[x] Dokumentasi hasil eksperimen di notebook dan artifact Kaggle
[x] README ringkas dan dokumentasi dataset
```

`Dataset-metadata.xlsx` telah diperiksa dan bersifat deskriptif; file tersebut tidak menyediakan pemetaan per-crop ke citra UAV sumber. Notebook menyediakan split stratified yang reproducible sebagai fallback, serta `GROUP_MAPPING_PATH` untuk menjalankan group split tanpa overlap ketika pemetaan `relative_path,group_id` sudah tersedia. Karena itu, hasil split fallback belum boleh diklaim bebas data leakage pada laporan akhir.

Tiga eksperimen telah selesai dijalankan pada Kaggle GPU. Partial Fine-Tuning dan Scratch mencapai validation accuracy `1.0000`; Partial Fine-Tuning mencapai hasil tersebut pada epoch 1 dan dipilih sebagai model utama. Blind test anonim berisi 20 citra per kelas menghasilkan akurasi internal `1.0000`. Latency Partial Fine-Tuning tercatat `3.157 ms/image`. Artifact eksperimen disimpan di `results/` pada repository setelah proses training.

Tahap berikutnya yang bersifat opsional adalah memperoleh group mapping dari sumber dataset dan memverifikasi perbedaan jumlah kelas antara folder citra dan teks metadata. Hasil saat ini tetap harus dilaporkan sebagai image-level split, bukan sebagai evaluasi yang bebas data leakage.

## 18. Kode Acuan dan Referensi Implementasi

Implementasi kode pada project ini akan **mengikuti struktur, alur eksperimen, dan pendekatan implementasi dari notebook `IJCCS_FruitAndVeg_resnet18.ipynb`** yang terdapat pada:

```text
materi_p2/
└── IJCCS_FruitAndVeg_resnet18.ipynb
```

Notebook tersebut digunakan sebagai **kode acuan (reference implementation)** untuk penerapan Transfer Learning menggunakan **ResNet18**.

Dengan demikian, project ini tidak membuat pipeline Transfer Learning dari awal, tetapi melakukan adaptasi terhadap kode acuan tersebut agar sesuai dengan dataset dan permasalahan klasifikasi tanaman UAV yang digunakan.

### Prinsip Adaptasi

Bagian-bagian yang menjadi acuan dari `IJCCS_FruitAndVeg_resnet18.ipynb` meliputi:

- penggunaan framework **PyTorch**;
- penggunaan model **ResNet18**;
- penggunaan pretrained weights dari **ImageNet**;
- preprocessing dan transformasi citra;
- mekanisme training dan validation;
- konfigurasi optimizer dan learning rate;
- penggunaan learning-rate scheduler;
- pencatatan loss dan accuracy;
- evaluasi performa model;
- visualisasi hasil training;
- serta pola implementasi eksperimen Transfer Learning.

Sedangkan bagian yang perlu disesuaikan dengan project ini meliputi:

- lokasi dan struktur dataset;
- jumlah dan nama kelas;
- pembacaan `Dataset-metadata.xlsx`;
- mekanisme train-validation split;
- penanganan potensi data leakage pada citra UAV;
- konfigurasi classification head untuk dua kelas:
  - Durian (_Durio zibethinus_)
  - Papaya (_Carica papaya_);

- serta konfigurasi eksperimen Feature Extraction, Partial Fine-Tuning, dan Training from Scratch.

Secara konseptual:

```text
IJCCS_FruitAndVeg_resnet18.ipynb
              │
              │  kode acuan
              ▼
     Adaptasi Pipeline
              │
              ├── Dataset InterDuPa-UAV
              ├── Metadata Analysis
              ├── Leakage Prevention
              └── 2-Class Classification
              │
              ▼
       ResNet18 Experiments
              │
       ┌──────┼─────────┐
       ▼      ▼         ▼
    Feature  Partial   Scratch
   Extraction   FT
```

### Aturan Implementasi

Agar eksperimen tetap konsisten dengan materi P2, implementasi baru akan mempertahankan pola kode dari notebook acuan sejauh masih relevan.

Perubahan kode hanya dilakukan apabila diperlukan untuk:

1. menyesuaikan dataset InterDuPa-UAV;
2. menyesuaikan jumlah kelas;
3. mencegah data leakage;
4. menjalankan eksperimen pada environment Kaggle;
5. membandingkan tiga strategi training;
6. dan menghasilkan evaluasi yang dibutuhkan oleh project.

Dengan pendekatan ini, `IJCCS_FruitAndVeg_resnet18.ipynb` berfungsi sebagai **baseline implementasi**, sedangkan `notebook-p2-vision-dan-deep.ipynb` merupakan **adaptasi dan pengembangan baseline tersebut untuk dataset tanaman dari citra UAV**.

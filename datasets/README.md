# Dataset InterDuPa-UAV

Dataset yang digunakan pada project ini adalah **InterDuPa-UAV: A UAV-based
Dataset for the Classification of Intercropped Durian and Papaya Trees**.
Dataset diperoleh dari Zenodo:

<https://zenodo.org/records/15664908>

Dataset berisi citra tanaman yang diambil menggunakan UAV pada area pertanian
dan dikelompokkan ke dalam dua kelas:

| Kelas | Jumlah citra |
| --- | ---: |
| Durian (`Durio zibethinus`) | 3.327 |
| Papaya (`Carica papaya`) | 2.872 |
| **Total** | **6.199** |

Metadata sumber yang disediakan dataset tersedia sebagai `Dataset-metadata.xlsx`.
File tersebut bersifat deskriptif dan tidak menyediakan pemetaan setiap crop
ke citra UAV sumber.

Repository juga menyediakan [`metadata.csv`](metadata.csv). File ini adalah
manifest yang dibuat dari folder kelas lokal, dengan kolom:

```text
relative_path,label,class_name
```

Manifest tersebut berisi 6.199 citra: 3.327 Durian dan 2.872 Papaya. File ini
berguna untuk memeriksa daftar file dan label, tetapi bukan metadata sumber UAV
dan tidak dapat digunakan untuk membuat `group_id` tanpa informasi tambahan.
Notebook saat ini tetap membaca citra dari folder kelas; `metadata.csv` berfungsi
sebagai manifest pemeriksaan dan tidak menjadi input wajib pipeline training.

Jumlah citra pada folder lokal yang digunakan notebook adalah 3.327 Durian dan
2.872 Papaya. Teks abstrak di file metadata mencantumkan angka tersebut dalam
urutan terbalik. Karena itu, folder citra dan label yang dibaca notebook menjadi
acuan eksperimen saat ini; perbedaan ini sebaiknya diverifikasi terhadap sumber
dataset sebelum pelaporan final.

## Mengapa dataset tidak disimpan di GitHub?

File citra berukuran besar sehingga tidak praktis dan tidak sesuai untuk
disimpan langsung di repository GitHub. `Dataset-metadata.xlsx` juga tidak
diikutkan karena merupakan file sumber lokal. File `metadata.csv` tetap
disimpan karena ukurannya kecil dan dibutuhkan sebagai manifest dataset.

## Cara menyiapkan dataset

1. Unduh dataset dari link Zenodo di atas.
2. Ekstrak atau salin folder kelas ke direktori `datasets/`.
3. Pastikan struktur minimalnya seperti berikut:

```text
datasets/
├── Dataset-metadata.xlsx
├── metadata.csv
├── Durian (durio zibethinus)/
│   ├── 0.jpg
│   └── ...
└── Papaya (carica papaya)/
    ├── 0.jpg
    └── ...
```

Dataset lokal juga dapat memiliki folder citra UAV sumber seperti `25 meter/`
dan `30 meter/`. Folder tersebut tidak dibaca sebagai kelas oleh notebook
klasifikasi saat ini.

Notebook utama `notebook-p2-vision-dan-deep.ipynb` merupakan notebook Kaggle
yang sudah menyimpan output eksperimen. Notebook akan menggunakan dataset
Kaggle ketika path `/kaggle/input/...` tersedia, dan akan menggunakan
`datasets/` secara otomatis ketika dijalankan di luar Kaggle. Pada Kaggle,
lokasi dataset diatur melalui `DATASET_ROOT`.

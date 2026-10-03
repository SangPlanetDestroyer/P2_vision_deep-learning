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

Metadata deskriptif dataset tersedia sebagai `Dataset-metadata.xlsx`.

Jumlah citra pada folder lokal yang digunakan notebook adalah 3.327 Durian dan
2.872 Papaya. Teks abstrak di file metadata mencantumkan angka tersebut dalam
urutan terbalik. Karena itu, folder citra dan label yang dibaca notebook menjadi
acuan eksperimen saat ini; perbedaan ini sebaiknya diverifikasi terhadap sumber
dataset sebelum pelaporan final.

## Mengapa dataset tidak disimpan di GitHub?

File citra berukuran besar sehingga tidak praktis dan tidak sesuai untuk
disimpan langsung di repository GitHub. Oleh karena itu, file citra mentah
tidak diikutkan dalam repository dan direktori dataset diatur agar diabaikan
oleh Git. README ini tetap disimpan sebagai dokumentasi sumber dataset.

## Cara menyiapkan dataset

1. Unduh dataset dari link Zenodo di atas.
2. Ekstrak atau salin folder kelas ke direktori `datasets/`.
3. Pastikan struktur minimalnya seperti berikut:

```text
datasets/
├── Dataset-metadata.xlsx
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

Notebook utama akan menggunakan `datasets/` secara otomatis ketika dijalankan
di luar environment Kaggle. Pada Kaggle, lokasi dataset diatur melalui
`DATASET_ROOT`.

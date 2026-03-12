# Smart Parking Detection System
Sistem deteksi parkir berbasis computer vision menggunakan edge computing pada NVIDIA Jetson Nano.

## Deskripsi
System ini mendeteksi slot parkir kosong dan terisi secara realtime menggunakan kamera USB, model YOLO yang dioptimasi dengan TensorRT, dan menampilkan hasilnya di LCD 1602 yang terpasang langsung pada Jetson Nano.

## Hardware yang Digunakan
- **NVIDIA Jetson Nano** — edge computing device utama
- **Kamera USB** — input video realtime
- **LCD 1602 I2C** — display output informasi parkir (alamat I2C: `0x27`)
- **Breadboard + Kabel Jumper (Male to Female)** — koneksi LCD ke Jetson
- **MicroSD Card** — storage OS dan model

## Software Stack
| Komponen | Versi | Fungsi |
| :--- | :--- | :--- |
| **JetPack** | R32.7.6 | OS dan driver NVIDIA untuk Jetson Nano |
| **Python** | 3.6.9 (sistem) | Runtime utama, wajib pakai `/usr/bin/python3` |
| **CUDA** | 10.2 | GPU acceleration |
| **TensorRT** | 8.2.1.8 | Optimasi dan inference model di GPU |
| **pycuda** | 2020.1 | Interface Python ke CUDA |
| **OpenCV** | 4.1.1 | Capture kamera dan visualisasi |
| **NumPy** | 1.13.3 | Operasi array untuk preprocessing |
| **smbus2** | 0.6.0 | Komunikasi I2C untuk LCD |
| **Miniforge/conda** | 26.1.0 | Diinstall tapi tidak digunakan untuk inference |

## Dataset
Model dilatih menggunakan dataset dari Kaggle:
🔗 **[Parking Lot Dataset (Kaggle)](https://www.kaggle.com/datasets/blanderbuss/parking-lot-dataset)**

## Alur Pipeline
### Training (Laptop RTX 3050)
```text
Dataset parkir → YOLO11 → model.pt → export → model.onnx
```

### Deployment (Jetson Nano)
```text
model.onnx → trtexec --fp16 → model.engine (TensorRT)
Kamera USB → preprocess → TensorRT inference → postprocess → hasil deteksi
Hasil deteksi → LCD 1602 (jumlah kosong/penuh)
Hasil deteksi → OpenCV window (bounding box + label)
```

## Model
- **Arsitektur:** YOLO11 (Ultralytics)
- **Format deploy:** TensorRT Engine (`.engine`) dengan optimasi FP16
- **Input shape:** `[1, 3, 480, 480]`
- **Output shape:** `[1, 6, 4725]`
- **Classes:** `empty` (slot kosong), `occupied` (slot terisi)
- **Confidence threshold:** `0.5`
- **IOU threshold:** `0.4`

## Catatan Penting Deployment
- Wajib gunakan `/usr/bin/python3` (Python 3.6 sistem), bukan conda environment, karena TensorRT binding hanya tersedia untuk Python 3.6 di JetPack 4.x
- `onnxruntime-gpu` tidak kompatibel karena membutuhkan GLIBC 2.29 sedangkan JetPack 4.x hanya punya GLIBC 2.27 — solusinya adalah TensorRT langsung
- Konversi ONNX ke TensorRT engine dilakukan sekali di Jetson menggunakan `trtexec`

## Ide Fitur Website Monitoring (Belum Diimplementasi)
Rencana pengembangan selanjutnya adalah menambahkan web dashboard untuk monitoring parkir yang bisa diakses dari internet. Arsitektur yang direncanakan:

**Stack:**
- **Flask** — web server ringan yang dijalankan langsung di Jetson Nano
- **ngrok** — tunnel agar web server lokal bisa diakses dari internet tanpa VPS
- **HTML/CSS/JavaScript** — frontend dashboard sederhana dengan auto-refresh

**Fitur yang direncanakan:**
- Halaman web realtime menampilkan jumlah slot kosong dan terisi
- Auto-refresh setiap beberapa detik
- Accessible dari perangkat manapun via link ngrok
- Tidak membutuhkan server eksternal atau konfigurasi router

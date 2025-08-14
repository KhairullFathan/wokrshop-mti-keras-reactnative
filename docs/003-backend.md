# 3. Pengembangan Backend dengan Python

## 3.1. Persiapan Lingkungan Pengembangan

### 3.1.1. Verifikasi Instalasi Python

Pastikan Python sudah terinstall di sistem Anda:

```bash
python --version
# atau
python3 --version
```

Jika belum terinstall, download dari [python.org](https://www.python.org/downloads/)

### 3.1.2. Inisialisasi Project

Buat folder project dan virtual environment:

```bash
# Membuat folder project
D:                                                # masuk ke partisi D
mkdir digit-recognition && cd digit-recognition   # membuat directory digit-recognition dan masuk ke directory
mkdir backend && cd backend                       # membuat directory backend dan masuk ke directory


python -m venv venv                               # membuat virtual environment

# Aktifkan virtual environment
.\venv\Scripts\activate                           # untuk Windows:

source venv/bin/activate                          # untuk Linux/Mac:
```

### 3.1.3. Install Dependencies

Install package yang diperlukan:

```bash
pip install tensorflow==2.19.0 Flask==3.1.1 Pillow==11.3.0 flask_cors
```

Buat file `requirements.txt` untuk menyimpan daftar dependencies:

```bash
pip freeze > requirements.txt
```

---

## 3.2. Struktur Project

```
digit-recognition/
├── backend/                  # Backend service
│   ├── venv/                 # Python virtual environment
│   ├── static/               # Direktori flask untuk menyimpan file statis
│   │   └── uploads/          # Direktori upload dari sisi client
│   ├── app.py                # Flask application
│   └── model.keras           # Trained model
└── frontend/                 # Frontend application
```

---

## 3.3. Implementasi Flask Backend

### 3.3.1. Import Library

Sebelum memulai pengembangan backend, kita perlu mengimpor berbagai library yang akan digunakan.

```python
import os                                                # library untuk interaksi dengan filesystem
import io                                                # library untuk penanganan stream I/O
import time                                              # library untuk penanganan waktu

import tensorflow as tf                                  # library machine learning
import numpy as np                                       # library komputasi numerik

from flask import Flask, request, json, jsonify, url_for # framework flask untuk webservice
from flask_cors import CORS                              # library flask tambahan untuk handle Cross-Origin

from PIL import Image                                    # library untuk proses gambar
```

### 3.3.2. Public Variabel

Pada tahap ini, kita akan mendefinisikan beberapa variabel publik yang digunakan sebagai konfigurasi awal program. Variabel ini mencakup daftar kelas yang akan dikenali oleh model, jumlah total kelas, pemanggilan model yang telah dilatih sebelumnya, serta penentuan ekstensi file gambar yang diizinkan untuk diproses.

```python
CLASS = ['0','1','2','3','4','5','6','7','8','9']   # daftar kelas yang dikenali model
CLASS_LEN = len(CLASS)                              # jumlah total kelas
MODEL = tf.keras.models.load_model('model.keras')   # load model yang telah dilatih sebelumnya
ALLOWED_EXT = set(['jpg', 'png', 'jpeg'])           # daftar extension file yang diizinkan
```

### 3.3.3. Inisialisasi Flask APP & Endpoint `/`

Pada bagian ini dilakukan inisialisasi Flask App yang akan berfungsi sebagai server utama untuk menangani permintaan (request) dari pengguna. Selain itu, diaktifkan juga fitur CORS (Cross-Origin Resource Sharing) agar API dapat diakses dari berbagai domain. Selanjutnya, dibuat endpoint root (/) yang akan menampilkan pesan sederhana sebagai indikasi bahwa Digit Recognition API telah berjalan.

```python
app = Flask(__name__)                                         # membuat objek `app` yang merupakan instance dari Flask
CORS(app)                                                     # mengaktifkan fitur CORS

@app.route('/')                                               # mendefinisikan route `/` (root) secara default methods = ['GET']
def home():                                                   # inisialisasi fungsi `home()`, yang dijalankan ketika mengakses endpoint `/`
  return jsonify({'message':'Digit Recognition API'}), 200    # mengembalikan respon dalam bentuk json dengan http status code 200

if __name__ == '__main__':
  port = 5000                                                 # inisialisasi port
  print(f"Server sedang berjalan di http://0.0.0.0:${port}")  # cetak console
  app.run(debug=True, host='0.0.0.0', port=port)              # running flask app
```

### 3.3.4. Menjalankan Aplikasi Flask & Testing request `/`

Setelah proses inisialisasi dan pembuatan endpoint selesai, langkah selanjutnya adalah menjalankan aplikasi Flask. Aplikasi dapat dijalankan dengan dua cara, yaitu menggunakan perintah flask run atau langsung mengeksekusi file Python yang berisi kode program.

```bash
flask run
# atau
python app.py
```

Setelah aplikasi berjalan, langkah selanjutnya mencoba untuk testing endpoint `/`, untuk mengetahui aplikasi telah berjalan dengan baik.

```bash
curl -X GET  http://localhost:5000/predict
```

### 3.3.5. Endpoint `predict/`

Pada bagian ini dibuat endpoint /predict yang berfungsi untuk menerima file gambar dari pengguna melalui metode POST, kemudian memprosesnya menggunakan model yang telah dilatih untuk menghasilkan prediksi digit. Endpoint ini juga mengembalikan hasil klasifikasi beserta persentase probabilitasnya dalam format JSON.

```python
@app.route('/predict', methods=['POST'])                  # mendefinisikan route `/predict` dengan methods ['POST']
def predict():                                            # inisialisasi fungsi `predict()`, yang dijalankan ketika mengakses endpoint `/predict`
  if 'file' not in request.files:                         # mengecek pakah request dari client mengirimkan file dengan nama 'file'
    return jsonify({'error': 'No file uploaded'}), 400    # mengembalikan respon json

  file = request.files['file']                            # file yang dikirimkan dari client disimpan pada variabel 'file'
  fileExtension = file.filename.split('.')[-1].lower()    # mengambil extension file

  if not (fileExtension in ALLOWED_EXT):                  # mengecek apakah extension file diizinkan
    return jsonify({"error":"Not allowed file extension."}), 400 # mengembalikan respon json

  newName = f"{time.time_ns()}.{fileExtension}"           # membuat nama unik dari timestamp
  imagePath = f"./static/uploads/{newName}"               # inisialisasi path penyimpanan
  file.save(imagePath)                                    # save file upload ke dalam path dan nama baru

  if not (os.path.exists(imagePath)):                     # memeriksa file yang di upload benar-benar berhasil tersimpan
    return jsonify({"error":"File not uploaded"}),404     # mengembalikan respon json jika file tidak ada

  try:
    image = Image.open(imagePath).convert('RGB')          # membuka file sesuai dengan path dan convert ke mode RGB
    image = image.resize((90, 140))                       # resize ukuran agar sesuai dengan model cnn
    imgArray = np.array(image) / 255.0                    # normalisasi
    imgArray = np.expand_dims(imgArray, axis=0)           # menambahkan dimensi agar sesuai dengan input model

    predictions = model.predict(imgArray)                 # fungsi prediksi dengan model .keras
    dictResult = {i: classes[i] for i in range(classLen)} # inisialisasi dictionary untuk mapping label kelas

    res = predictions[0]                                  # mengambil hasil prediksi indeks [0]

    sortedIndex = res.argsort()[::-1]                     # melakukan sort indeks kelas berdasarkan probabilitas tertinggi
    prob = res[sortedIndex[:classLen]]                    # melakukan probabilitas hasil prediksi sesuai urutan

    classificationResult = []                             # inisialisasi list hasil klasifikasi

    for i in range(classLen):                             # looping sebanyak jumlah kelas
      # membuat objek sementara untuk menyimpan nama kelas dan probabilitas
      resultDict = {
        'class': dictResult[sortedIndex[i]],              # mengambil label kelas
        'prob': round(float(prob[i] * 100), 2)            # mengambil probabilitas, di konversi ke dalam persen (%) dan dibulatkan hanya mengambil 2 angka desimal
      }
      classificationResult.append(resultDict)             # menambahkan objek 'resultDict' ke list hasil klasifikasi

    # mengembalikan respon dalam bentuk json
    return jsonify({
      'classification_result': classificationResult,
    }), 200

  except Exception as e:                                  # mengembalikan error jika terjadi kesalahan pada block try
    return jsonify({'error': str(e)}), 500                # mengembalikan pesan error dengan kode 500
```

### 3.3.6. Testing request `predict/`

```bash
curl -X POST -F "file=@D:\0004DEV\008Experimen\DigitHadwriting\dataset\2\Two_full (1).jpg" http://localhost:5000/predict
```

### 3.3.7. Kode Lengkap `app.py`

```python
# import library
import os
import io
import time
import tensorflow as tf
import numpy as np
from flask import Flask, request, json, jsonify, url_for
from flask_cors import CORS
from PIL import Image

# Public Variabel
CLASS = ['0','1','2','3','4','5','6','7','8','9']
CLASS_LEN = len(CLASS)
MODEL = tf.keras.models.load_model('model.keras')
ALLOWED_EXT = set(['jpg', 'png', 'jpeg'])

# Inisialisasi Flask APP & Endpoint '/'
app = Flask(__name__)
CORS(app)

@app.route('/')
def home():
  return jsonify({'message':'Digit Recognition API'}), 200

# Endpoint 'predict/'
@app.route('/predict', methods=['POST'])
def predict():
  if 'file' not in request.files:
    return jsonify({'error': 'No file uploaded'}), 400

  file = request.files['file']
  fileExtension = file.filename.split('.')[-1].lower()

  if not (fileExtension in ALLOWED_EXT):
    return jsonify({"error":"Not allowed file extension."}), 400

  newName = f"{time.time_ns()}.{fileExtension}"
  imagePath = f"./static/uploads/{newName}"
  file.save(imagePath)

  if not (os.path.exists(imagePath)):
    return jsonify({"error":"File not uploaded"}),404

  try:
    image = Image.open(imagePath).convert('RGB')
    image = image.resize((90, 140))
    imgArray = np.array(image) / 255.0
    imgArray = np.expand_dims(imgArray, axis=0)

    predictions = MODEL.predict(imgArray)
    dictResult = {i: CLASS[i] for i in range(CLASS_LEN)}

    res = predictions[0]

    sortedIndex = res.argsort()[::-1]
    prob = res[sortedIndex[:CLASS_LEN]]

    classificationResult = []

    for i in range(CLASS_LEN):
      resultDict = {
        'class': dictResult[sortedIndex[i]],
        'prob': round(float(prob[i] * 100), 2)
      }
      classificationResult.append(resultDict)

    return jsonify({
      'classification_result': classificationResult,
      'image_url': url_for('static', filename=f'uploads/{newName}', _external=True)
    }), 200

  except Exception as e:
    return jsonify({'error': str(e)}), 500


if __name__ == '__main__':
  port = 5000
  print(f"Server sedang berjalan di http://0.0.0.0:{port}")
  app.run(debug=True, host='0.0.0.0', port=port)

```

# 1. Penyiapan dataset

## 1.1. Sumber Dataset

Dataset yang digunakan berasal dari platform Kaggle:  
🔗 [Handwritten Digits 0-9 Dataset](https://www.kaggle.com/datasets/olafkrastovski/handwritten-digits-0-9)

---

## 1.2 Deskripsi Dataset

Dataset ini berisi gambar angka tulisan tangan dari 0 hingga 9 dengan karakteristik khusus:

- **Jumlah Data**: Sekitar 21.600 gambar angka
- **Format**: Gambar berwarna (full color) dalam format .jpg
- **Dimensi**: 90x140 piksel
- **Asal Notasi**: Menggunakan notasi angka Eropa (Swiss)

---

## 1.3 Keunikan Dataset

Dataset ini memiliki beberapa ciri khas yang membedakannya dari MNIST:

🎯 **Notasi Eropa**:

- Khususnya angka **1** dan **7** memiliki gaya penulisan yang berbeda dengan notasi Amerika
- Beberapa angka mungkin memiliki **border hitam kecil** di sekelilingnya

🖼️ **Format Gambar**:

- Berwarna (RGB) dibanding MNIST yang grayscale
- Resolusi lebih besar (90x140px) dibanding MNIST (28x28px)

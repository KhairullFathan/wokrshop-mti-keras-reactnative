# 2. Proses Training dan Testing Dataset di Google Colab

## 2.1. Persiapan Data

### 2.1.1. Load dan Ekstrak Dataset dari Google Drive

Dataset dalam format `.zip` diakses langsung dari Google Drive.

```python
from google.colab import drive
import zipfile

# Mount Google Drive
drive.mount('/content/drive')

# Path dataset di Google Drive
path = "digit-handwritten"
dataPath = f"/content/drive/MyDrive/workshop/{path}.zip"

# Ekstrak dataset
zip_ref = zipfile.ZipFile(dataPath, 'r')
zip_ref.extractall(path)  # Ekstrak ke direktori root Colab
zip_ref.close()

print(f"Dataset berhasil diekstrak ke folder: {path}")
```

### **Catatan**:

- Pastikan file zip sudah ada di Google Drive sebelum menjalankan kode.
- Jika file zip berada dalam folder tertentu di Drive, pastikan `dataPath` sesuai, misalnya jika berada dalam folder `workshop`:
  ```python
  dataPath = f"/content/drive/MyDrive/workshop/{path}.zip"
  ```
- Jika ingin menyimpan hasil ekstraksi di folder tertentu di Drive, gunakan:
  ```python
  zip_ref.extractall(f"/content/drive/MyDrive/target_folder/{path}")
  ```

### 2.1.2. Split Data (Train, Validation, Test)

Menggunakan `split_folders` untuk membagi dataset menjadi **train (70%)**, **validation (20%)**, dan **test (10%)**.

```python
!pip install split-folders
import splitfolders

# Split dataset
splitfolders.ratio(
    path,                      # Folder dataset asli di yang berada di root google colab
    output="split_dataset",    # Folder output hasil split
    seed=42,                   # Random seed untuk reproducibility
    ratio=(0.7, 0.2, 0.1),     # Rasio train, val, test
    group_prefix=None          # Tidak mengelompokkan berdasarkan prefix
)
```

---

## 2.2. Persiapan Data dengan `ImageDataGenerator`

Menggunakan `ImageDataGenerator` untuk augmentasi data dan normalisasi.

```python
from tensorflow.keras.preprocessing.image import ImageDataGenerator

# Konfigurasi ukuran gambar
img_height, img_width = 140, 90  # Sesuaikan dengan dimensi dataset atau input layer pada CNN
BATCH_SIZE = 32

def create_data(directory):
    # Augmentasi untuk data training
    train_datagen = ImageDataGenerator(
        rescale=1.0 / 255,           # Normalisasi pixel [0, 1]
        rotation_range=40,           # Rotasi gambar ±40 derajat
        width_shift_range=0.2,       # Geser horizontal 20%
        height_shift_range=0.2,      # Geser vertikal 20%
        shear_range=0.2,             # Shear transformasi
        zoom_range=0.2,              # Zoom in/out 20%
        horizontal_flip=True,        # Flip horizontal
        fill_mode='nearest'          # Isi pixel kosong dengan nearest
    )

    # Hanya normalisasi untuk validation & test (tanpa augmentasi)
    val_test_datagen = ImageDataGenerator(rescale=1.0 / 255)

    # Generator untuk data training
    train_ds = train_datagen.flow_from_directory(
        f"./{directory}/train",
        target_size=(img_height, img_width),
        class_mode='categorical',
        batch_size=BATCH_SIZE,
        shuffle=True
    )

    # Generator untuk data validation
    val_ds = val_test_datagen.flow_from_directory(
        f"./{directory}/val",
        target_size=(img_height, img_width),
        class_mode='categorical',
        batch_size=BATCH_SIZE,
        shuffle=True
    )

    # Generator untuk data testing
    test_ds = val_test_datagen.flow_from_directory(
        f"./{directory}/test",
        target_size=(img_height, img_width),
        class_mode='categorical',
        batch_size=BATCH_SIZE,
        shuffle=False  # Tidak di-shuffle untuk evaluasi
    )

    return train_ds, val_ds, test_ds

# Panggil fungsi untuk membuat dataset
train_ds, val_ds, test_ds = create_data("split_dataset")
```

---

## 2.3. Membangun Model Sederhana untuk Training

Menggunakan **CNN (Convolutional Neural Network)** dengan arsitektur dasar.

```python
import tensorflow as tf
from tensorflow.keras import layers, models

# Membangun model CNN
model = models.Sequential([
    # Blok Convolutional 1
    layers.Conv2D(32, (3, 3), activation='relu', input_shape=(img_height, img_width, 3)),
    layers.MaxPooling2D((2, 2)),

    # Blok Convolutional 2
    layers.Conv2D(64, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),

    # Blok Convolutional 3
    layers.Conv2D(128, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),

    # Flatten untuk Fully Connected Layer
    layers.Flatten(),

    # Dense Layers
    layers.Dense(256, activation='relu'),
    layers.Dropout(0.5),  # Dropout untuk mengurangi overfitting
    layers.Dense(10, activation='softmax')  # Output layer (10 kelas angka 0-9)
])

# Kompilasi model
model.compile(
    optimizer='adam',
    loss='categorical_crossentropy',
    metrics=['accuracy']
)

# Ringkasan model
model.summary()
```

---

## 2.4. Training Model

Melatih model menggunakan `model.fit()`.

```python
# Training model
history = model.fit(
    train_ds,
    validation_data=val_ds,
    epochs=50,  # Jumlah epoch maksimum
)
```

### Plot Hasil Training

```python
import matplotlib.pyplot as plt

acc = history.history['accuracy']
loss = history.history['loss']
valAcc = history.history['val_accuracy']
valLoss = history.history['val_loss']

epochs = range(len(acc))
# show image
fig, axs = plt.subplots(1, 2, figsize=(14, 6))

# Accuracy
axs[0].plot(epochs, acc, label='Training Accuracy')
axs[0].plot(epochs, valAcc, label='Validation Accuracy')
axs[0].set_xlabel("Epochs")
axs[0].set_ylabel('Model Accuracy')
axs[0].legend()

# Loss
axs[1].plot(epochs, loss, label='Training Loss')
axs[1].plot(epochs, valLoss, label='Validation Loss')
axs[1].set_xlabel("Epochs")
axs[1].set_ylabel('Model Loss')
axs[1].legend()
plt.show()
```

---

## 2.5. Evaluasi Model

Menguji model dengan data testing.

```python
# Evaluasi model
test_loss, test_acc = model.evaluate(test_ds)
print(f"Test Accuracy: {test_acc * 100:.2f}%")
```

## 2.6. Simpan Model dalam format `.keras`

Model yang sudah dilatih disimpan dalam format `nama-model.keras` sehingga dapat diimplementasikan pada aplikasi mobile android.

```python
from google.colab import files
# Save model
model.save("handwritten_digits_model.keras")
# Download model
files.download("handwritten_digits_model.keras")
```

🎉 **Model siap digunakan untuk prediksi angka tulisan tangan!** 🚀

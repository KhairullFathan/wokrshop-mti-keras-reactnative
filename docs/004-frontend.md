# 4. Pengembangan Frontend dengan React Native

## 4.1. Persiapan Lingkungan Pengembangan

### 4.1.1. Verifikasi Instalasi nodejs

Pastikan Python sudah terinstall di sistem Anda:

```bash
node --version
# atau
node -v
```

Jika belum terinstall, download dari [nodejs.org](https://nodejs.org/en/download)

### 4.1.2. Inisialisasi Project

Langkah pertama dalam membangun aplikasi frontend adalah membuat struktur folder project dan menginisialisasi project Expo. Pada tahap ini, kita akan membuat folder khusus untuk frontend, lalu menjalankan perintah untuk membuat project Expo di dalamnya.

```bash
D:                                  # masuk ke partisi D
cd digit-recognition                # masuk ke directory digit-recognition
mkdir frontend && cd frontend       # membuat directory frontend dan masuk ke directory frontend

npx create-expo-app@latest .        # membuat project expo di directory `.` (terkini dalam hal ini frontend)
```

Proses `npx create-expo-app` akan memakan waktu beberapa menit, tergantung pada **kecepatan koneksi internet** dan **spesifikasi perangkat** yang digunakan, karena perintah ini akan mengunduh dependensi dan mengonfigurasi struktur dasar aplikasi. Setelah proses selesai, folder project akan berisi file dan direktori bawaan yang siap digunakan untuk pengembangan aplikasi **React Native** berbasis Expo.

Secara default, **Expo** akan menghasilkan beberapa direktori dan file bawaan sebagai dokumentasi dan contoh penggunaan. Jika ingin memulai dari struktur project yang lebih bersih, Anda dapat menggunakan perintah:

```bash
npm run reset-project
```

Perintah ini akan memindahkan file dan direktori contoh yang tidak diperlukan ke dalam directory `app-example`, sehingga project menjadi lebih rapi dan siap untuk dikembangkan sesuai kebutuhan aplikasi yang akan dibuat.

### 4.1.3. Install library tambahan yang diperlukan

Tahap ini bertujuan untuk menginstal library tambahan yang akan digunakan pada aplikasi frontend. Masing-masing library memiliki fungsi tertentu, mulai dari melakukan HTTP request hingga menampilkan grafik. Berikut perintah dan penjelasannya:

```bash
npm install axios                         # Menginstal Axios, library untuk melakukan HTTP request (GET, POST, dll) ke backend API
npm expo-image-picker                     # Menginstal modul Expo untuk memilih gambar dari galeri
npx expo install react-native-svg         # Menginstal react-native-svg untuk manipulasi gambar berbasis SVG
npx expo install react-native-chart-kit   # Menginstal library untuk membuat grafik/chart pada React Native
npx expo install lodash                   # Menginstal Lodash, library utilitas untuk manipulasi data (array, objek, dll)
npx expo install @expo/ngrok              # Menginstal @expo/ngrok, library untuk tunnel agar bisa di expose ke jaringan public
```

### 4.1.4. Menjalankan Project Expo

Setelah semua library tambahan berhasil diinstal, langkah berikutnya adalah menjalankan project **Expo** untuk memastikan bahwa instalasi berhasil dan aplikasi dapat berjalan di perangkat pengembangan. Gunakan perintah berikut:

```bash
npm start
# atau
npx expo start

npx start --tunnel
# atau
npx expo start --tunnel
```

**Penjelasan:**

- **`npm start`** merupakan perintah untuk menjalankan script `start` yang sudah didefinisikan di `package.json` (biasanya memanggil `expo start`).
- **`npx expo start`** merupakan perintah untuk menjalankan perintah `expo start` langsung melalui **npx**, tanpa harus menginstalnya secara global.
- **`--tunnel`** merupakan opsi untuk membuat koneksi melalui ngrok tunnel, sehingga aplikasi dapat diakses dari jaringan internet luar (bukan hanya jaringan lokal).

Saat perintah ini dijalankan, Expo CLI akan:

1. Membuka **Metro Bundler** di terminal atau browser.
2. Menampilkan **QR Code** untuk memudahkan membuka aplikasi di perangkat fisik melalui **Expo Go**.
3. Memberikan opsi menjalankan aplikasi di **Android Emulator**, **iOS Simulator**, atau **browser (web)**.

---

## 4.2. Struktur Project

```
digit-recognition/
├── backend/                  # backend service (Flask API dan model machine learning)
├── frontend/                 # frontend application (React Native + Expo)
│   ├── app/                  # folder source code utama aplikasi Expo
│   │   ├── _layout.jsx       # template yang digunakan
│   │   └── index.jsx         # halaman default saat pertama kali
│   ├── assets/               # folder untuk menyimpan aset (gambar, ikon, font, dll)
│   ├── node_modules/         # folder berisi dependensi project frontend
│   ├── app.json              # file konfigurasi utama Expo
│   └── package.json          # file konfigurasi npm berisi daftar dependensi dan script
```

---

## 4.3. Frontend

### 4.3.1. `_layout.jsx`

File **`_layout.jsx`** pada Expo Router berfungsi sebagai **layout utama** atau **pembungkus halaman** di dalam folder yang sama. File ini akan secara otomatis digunakan oleh semua halaman (`.jsx` atau `.js`) yang berada dalam direktori tersebut. Dengan adanya `_layout.jsx`, kita dapat mengatur navigasi, tema, atau konfigurasi global untuk seluruh halaman di dalamnya.

```javascript
import { Stack } from 'expo-router'

export default function RootLayout() {
  return <Stack />
}
```

Lakukan sedikit modifikasi dengan tujuan untuk menambahkan konfigurasi khusus pada setiap halaman.

```javascript
import { Stack } from 'expo-router'

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ headerShown: false }} />
    </Stack>
  )
}
```

Kode `<Stack.Screen name="index" options={{ headerShown: false }} />` bertujuan untuk menambahkan konfigurasi pada halaman index misalnya `headerShown:false`, yaitu menghilangkan header default.

### 4.3.2. `index.jsx`

File `index.jsx` bertindak sebagai halaman utama (home screen) yang pertama kali ditampilkan saat aplikasi dijalankan. Pada tahap ini, komponen masih berisi tampilan dasar dengan teks bawaan dari template Expo, yang ditempatkan di tengah layar menggunakan properti flexbox pada React Native.

```jsx
import { Text, View } from 'react-native'

export default function Index() {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <Text>Edit app/index.tsx to edit this screen.</Text>
    </View>
  )
}
```

### 4.3.3. `components/Header.jsx`

Pada directory `app` anda bisa membuat directory baru dengan nama `components` yang merupakan directory yang digunakan untuk menyimpan semua komponen sehingga dapat digunakan kemabali jika diperlukan. file `Header.jsx` (Penamaan kompoenen diawali dengan huruf kapital) merupakan file yang berisikan komponen Header

```jsx
import { Text, View } from 'react-native'

export default function Header(props) {
  return (
    <View
      style={{
        backgroundColor: '#000',
        paddingVertical: 10,
        paddingHorizontal: 15,
      }}
    >
      <Text
        style={{
          color: '#fff',
          fontWeight: 900,
          fontSize: 16,
        }}
      >
        {props.title || ''}
      </Text>
    </View>
  )
}
```

### 4.3.4. `components/Hero.jsx`

Pada tahap ini, kita membuat **komponen `Hero.jsx`** yang berfungsi sebagai bagian tampilan pembuka pada aplikasi. Komponen ini menampilkan judul (_heading_) dan deskripsi singkat mengenai fungsi aplikasi, yaitu pendeteksian angka dari tulisan tangan menggunakan teknologi AI. Seluruh elemen menggunakan **`View`** sebagai pembungkus, dengan dua elemen **`Text`** untuk judul dan deskripsi, serta pengaturan gaya (_styling_) langsung di dalam komponen agar tampilan rapi, terpusat, dan mudah dibaca.

```jsx
import { Text, View } from 'react-native'

export default function Hero() {
  return (
    <View
      style={{
        alignItems: 'center',
        gap: 5,
        paddingVertical: 20,
        paddingHorizontal: 20,
      }}
    >
      <Text
        style={{
          fontSize: 28,
          textAlign: 'center',
        }}
      >
        Handwriting Digit Recognition
      </Text>
      <Text
        style={{
          fontSize: 14,
          textAlign: 'center',
          fontWeight: 300,
        }}
      >
        Use AI technology to recognize {'\n'}
        numbers from your handwriting {'\n'}
        quickly, accurately, and conveniently.
      </Text>
    </View>
  )
}
```

### 4.3.5. `components/Classification.jsx`

Pada bagian ini, kita membuat **komponen `Classification.jsx`** yang bertanggung jawab sebagai area unggah (_upload_) gambar tulisan tangan untuk proses deteksi angka. Komponen ini menggunakan **`useState`** untuk menyimpan status **loading** dan hasil prediksi (**predict**) yang berisi informasi gambar serta hasil klasifikasi. Tombol unggah dibuat menggunakan **`TouchableOpacity`** dengan gaya border _dashed_ untuk menandakan area _drop/upload_. Saat tombol ditekan, fungsi `pickImage()` akan dipanggil.

```jsx
import axios from 'axios'
import * as ImagePicker from 'expo-image-picker'
import { useState } from 'react'
import {
  ActivityIndicator,
  Dimensions,
  Image,
  Text,
  TouchableOpacity,
  View,
} from 'react-native'
import { PieChart } from 'react-native-chart-kit'

export default function Classification() {
  const [loading, setLoading] = useState(false)
  const [predict, setPredict] = useState({
    image: null,
    result: [],
  })

  function pickImage() {
    console.log('pilih image')
  }

  return (
    <View
      style={{
        paddingHorizontal: 50,
      }}
    >
      <Text
        style={{
          fontSize: 12,
          fontWeight: 500,
          color: '#aaa',
          marginTop: 10,
          textAlign: 'center',
        }}
      >
        Upload image here:
      </Text>
      <TouchableOpacity
        style={{
          display: 'flex',
          borderColor: '#000',
          borderStyle: 'dashed',
          borderRadius: 4,
          borderWidth: 2,
          marginBottom: 10,
        }}
        onPress={pickImage}
      >
        <View
          style={{
            paddingVertical: 30,
          }}
        >
          <Text style={{ textAlign: 'center' }}>Click to upload</Text>
        </View>
      </TouchableOpacity>
    </View>
  )
}
```

### 4.3.6 `Classification.pickImage`

```javascript
async function pickImage() {
  try {
    const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync()
    if (status !== 'granted') {
      alert('Permission to access gallery is required!')
      return
    }
    const imagePick = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: 'images',
      allowsEditing: true,
      quality: 1,
    })

    if (
      !imagePick ||
      imagePick.canceled ||
      !Array.isArray(imagePick.assets) ||
      imagePick.assets.length === 0
    ) {
      alert('No image selected')
      return
    }

    setLoading(true)
    const asset = imagePick.assets[0]

    const formData = new FormData()
    formData.append('file', {
      uri: asset.uri,
      name: asset.fileName || 'image.jpg',
      type: asset.mimeType,
    })

    let response
    response = await axios.post(
      'http//localhost:5000', // URL Backend Flask
      formData,
      {
        headers: { 'Content-Type': 'multipart/form-data' },
        timeout: 60000,
      }
    )

    const { image_url, classification_result } = response.data || {}
    if (!Array.isArray(classification_result)) {
      throw new Error('Invalid response format')
    }

    setPredict({
      image: asset.uri,
      result: classification_result,
    })
  } catch (error) {
    const errorMessage = error.response
      ? `Server error: ${error.response.status}`
      : error.message || 'Failed to process image'
    alert(errorMessage)
    setPredict({
      image: null,
      result: [],
    })
  } finally {
    setLoading(false)
  }
}
```

Fungsi **`pickImage`** pada komponen ini berperan untuk menangani seluruh proses pemilihan gambar dari galeri hingga mengirimkannya ke server backend untuk dilakukan klasifikasi angka tulisan tangan.

Langkah kerjanya sebagai berikut:

1. **Meminta izin akses galeri** menggunakan `ImagePicker.requestMediaLibraryPermissionsAsync()`. Jika izin tidak diberikan, proses dihentikan dengan menampilkan pesan peringatan.
2. **Membuka galeri** menggunakan `ImagePicker.launchImageLibraryAsync()` dengan pengaturan hanya mengizinkan file gambar, mendukung pengeditan, dan kualitas maksimal.
3. **Validasi hasil pemilihan gambar** memastikan bahwa pengguna benar-benar memilih file yang valid. Jika tidak, akan muncul peringatan _"No image selected"_.
4. **Menyiapkan data untuk dikirim** dengan membungkus gambar ke dalam objek `FormData`. Informasi yang dikirim mencakup `uri`, `name`, dan `type` file.
5. **Mengirim permintaan ke server Flask** menggunakan `axios.post()` dengan `Content-Type` `multipart/form-data`. Server akan mengembalikan `image_url` dan `classification_result`.
6. **Menyimpan hasil prediksi** ke dalam state `predict` jika data valid, termasuk URL gambar yang dipilih dan hasil klasifikasinya.
7. **Menangani kesalahan** baik dari sisi jaringan, server, maupun format respons yang tidak sesuai, lalu mengosongkan data prediksi jika terjadi error.
8. **Mengatur status loading** agar UI dapat menampilkan indikator proses selama unggah dan prediksi berlangsung.

**Catatan**
Lakukan import library yang diperlukan:

1. `import * as ImagePicker from 'expo-image-picker'`
2. `import axios from 'axios'`

### 4.3.7. `Classification.Result`

```jsx
function Result(props) {
  const { image, result } = props.prediction
  const screenWidth = Dimensions.get('window').width

  if (!image || !Array.isArray(result) || !(result.length > 0)) return null

  function getRandomColor() {
    const letters = '0123456789ABCDEF'
    let color = '#'
    for (let i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)]
    }
    return color
  }

  const chartSeries = result?.map((e, i) => ({
    name: `Class ${e.class}`,
    population: e.prob,
    color: getRandomColor(),
  }))

  return (
    <View>
      <Image
        source={{ uri: image }}
        style={{
          height: undefined,
          width: '100%',
          aspectRatio: 1,
          resizeMode: 'contain',
        }}
      />
      <View style={{ marginVertical: 20 }}>
        <Text style={{ textAlign: 'center' }}>Class Probability (%)</Text>
        <PieChart
          data={chartSeries}
          width={screenWidth - 50}
          height={220}
          chartConfig={{
            color: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`,
          }}
          accessor="population"
          backgroundColor="transparent"
          paddingLeft="15"
          absolute
          hasLegend
        />
      </View>
    </View>
  )
}
```

Fungsi **`Result`** digunakan untuk menampilkan **hasil prediksi klasifikasi angka tulisan tangan** dalam bentuk gambar asli yang diunggah pengguna, sekaligus **diagram pie** yang memperlihatkan persentase probabilitas untuk setiap kelas.

Alur kerjanya sebagai berikut:

1. **Mengambil dan memvalidasi data prediksi**

- Data `image` dan `result` diambil dari `props.prediction`.
- Jika gambar kosong (`!image`), `result` bukan array, atau array `result` kosong, maka fungsi langsung mengembalikan `null` dan tidak menampilkan apapun.

2. **Membuat fungsi warna acak**

   - Fungsi `getRandomColor()` menghasilkan warna acak dalam format heksadesimal (`#RRGGBB`) untuk membedakan tiap kelas pada pie chart.

3. **Memformat data untuk diagram pie**

- Array `result` diubah menjadi `chartSeries`, yang berisi:

  - `name` → label kelas, misalnya _Class 0_, _Class 1_, dll.
  - `population` → nilai probabilitas prediksi kelas tersebut.
  - `color` → warna acak yang dibuat dari `getRandomColor()`.

4. **Menampilkan gambar hasil unggahan**

- Menggunakan `Image` dengan `aspectRatio: 1` agar proporsi tidak berubah.
- `resizeMode: 'contain'` menjaga gambar tetap utuh tanpa terpotong.

5. **Menampilkan diagram pie probabilitas kelas**

- Menggunakan komponen `PieChart` untuk memvisualisasikan persentase probabilitas setiap kelas.
- Lebar pie chart disesuaikan dengan ukuran layar (`screenWidth - 50`).
- `absolute` dan `hasLegend` digunakan agar label dan nilai persentase ditampilkan jelas di luar chart.

Hasilnya, pengguna dapat melihat **gambar yang diprediksi** beserta **distribusi persentase probabilitas setiap kelas** secara visual dan informatif.

---

## 4.4. Kode Final

### 4.4.1. `_layout.jsx`

```jsx
import { Stack } from 'expo-router'

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ headerShown: false }} />
    </Stack>
  )
}
```

### 4.4.2. `index.jsx`

```jsx
import { ScrollView, View } from 'react-native'
import Classification from './components/Classification'
import Header from './components/Header'
import Hero from './components/Hero'

export default function Index() {
  return (
    <View
      style={{
        flex: 1,
        marginTop: 40,
      }}
    >
      <Header title={'Handwriting Recognition'} />
      <ScrollView>
        <Hero />
        <Classification />
      </ScrollView>
    </View>
  )
}
```

### 4.4.3. `components/Header.jsx`

```jsx
import { Text, View } from 'react-native'

export default function Header(props) {
  return (
    <View
      style={{
        backgroundColor: '#000',
        paddingVertical: 10,
        paddingHorizontal: 15,
      }}
    >
      <Text
        style={{
          color: '#fff',
          fontWeight: 900,
          fontSize: 16,
        }}
      >
        {props.title || ''}
      </Text>
    </View>
  )
}
```

### 4.4.4. `components/Hero.jsx`

```jsx
import { Text, View } from 'react-native'

export default function Hero() {
  return (
    <View
      style={{
        alignItems: 'center',
        gap: 5,
        paddingVertical: 20,
        paddingHorizontal: 20,
      }}
    >
      <Text
        style={{
          fontSize: 28,
          textAlign: 'center',
        }}
      >
        Handwriting Digit Recognition
      </Text>
      <Text
        style={{
          fontSize: 14,
          textAlign: 'center',
          fontWeight: 300,
        }}
      >
        Use AI technology to recognize {'\n'}
        numbers from your handwriting {'\n'}
        quickly, accurately, and conveniently.
      </Text>
    </View>
  )
}
```

### 4.4.5. `components/Classification.jsx`

```jsx
import axios from 'axios'
import * as ImagePicker from 'expo-image-picker'
import { useState } from 'react'
import {
  ActivityIndicator,
  Dimensions,
  Image,
  Text,
  TouchableOpacity,
  View,
} from 'react-native'
import { PieChart } from 'react-native-chart-kit'

export default function Classification() {
  const [loading, setLoading] = useState(false)
  const [predict, setPredict] = useState({
    image: null,
    result: [],
  })

  async function pickImage() {
    try {
      const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync()
      if (status !== 'granted') {
        alert('Permission to access gallery is required!')
        return
      }
      const imagePick = await ImagePicker.launchImageLibraryAsync({
        mediaTypes: 'images',
        allowsEditing: true,
        quality: 1,
      })

      if (
        !imagePick ||
        imagePick.canceled ||
        !Array.isArray(imagePick.assets) ||
        imagePick.assets.length === 0
      ) {
        alert('No image selected')
        return
      }

      setLoading(true)
      const asset = imagePick.assets[0]

      const formData = new FormData()
      formData.append('file', {
        uri: asset.uri,
        name: asset.fileName || 'image.jpg',
        type: asset.mimeType,
      })

      let response
      response = await axios.post(
        'http//localhost:5000', // URL Backend Flask
        formData,
        {
          headers: { 'Content-Type': 'multipart/form-data' },
          timeout: 60000,
        }
      )
      const { classification_result } = response.data || {}
      if (!Array.isArray(classification_result)) {
        throw new Error('Invalid response format')
      }

      setPredict({
        image: asset.uri,
        result: classification_result,
      })
      console.log(predict)
    } catch (error) {
      const errorMessage = error.response
        ? `Server error: ${error.response.status}`
        : error.message || 'Failed to process image'
      alert(errorMessage)
      setPredict({
        image: null,
        result: [],
      })
    } finally {
      setLoading(false)
    }
  }

  return (
    <View
      style={{
        paddingHorizontal: 50,
      }}
    >
      <Text
        style={{
          fontSize: 12,
          fontWeight: 500,
          color: '#aaa',
          marginTop: 10,
          textAlign: 'center',
        }}
      >
        Upload image here:
      </Text>
      <TouchableOpacity
        style={{
          display: 'flex',
          borderColor: '#000',
          borderStyle: 'dashed',
          borderRadius: 4,
          borderWidth: 2,
          marginBottom: 10,
        }}
        onPress={pickImage}
      >
        <View
          style={{
            paddingVertical: 30,
          }}
        >
          <Text style={{ textAlign: 'center' }}>Click to upload</Text>
        </View>
      </TouchableOpacity>
      {loading && <ActivityIndicator size={'large'} color={'#000'} />}
      {predict && predict.image && predict.result && (
        <Result prediction={predict} />
      )}
    </View>
  )
}

function Result(props) {
  const { image, result } = props.prediction
  const screenWidth = Dimensions.get('window').width

  if (!image || !Array.isArray(result) || !(result.length > 0)) return null

  function getRandomColor() {
    const letters = '0123456789ABCDEF'
    let color = '#'
    for (let i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)]
    }
    return color
  }

  const chartSeries = result?.map((e, i) => ({
    name: `Class ${e.class}`,
    population: e.prob,
    color: getRandomColor(),
  }))

  return (
    <View>
      <Image
        source={{ uri: image }}
        style={{
          height: undefined,
          width: '100%',
          aspectRatio: 1,
          resizeMode: 'contain',
        }}
      />
      <View style={{ marginVertical: 20 }}>
        <Text style={{ textAlign: 'center' }}>Class Probability (%)</Text>
        <PieChart
          data={chartSeries}
          width={screenWidth - 50}
          height={220}
          chartConfig={{
            color: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`,
          }}
          accessor="population"
          backgroundColor="transparent"
          paddingLeft="15"
          absolute
          hasLegend
        />
      </View>
    </View>
  )
}
```

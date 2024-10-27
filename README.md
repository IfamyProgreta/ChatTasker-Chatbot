
## 📂 **1. Nama Modul: ChatTasker-Chatbot**

- **Fungsi Utama**: Mengelola interaksi chatbot dengan pengguna untuk penanganan tugas, memberi respons terkait status tugas, menandai tugas sebagai selesai, dan mengatur pengingat.
- **Teknologi**:
  - *Framework Bot*: Dialogflow (atau API chatbot lain)
  - *Server Framework*: Node.js
  - *Database dan Penyimpanan*: Firebase
- **Repositori GitHub**: ChatTasker-Chatbot

---

## 📘 **2. Deskripsi Modul**

Modul Chatbot bertanggung jawab atas komunikasi langsung dengan pengguna menggunakan antarmuka percakapan. Dialog chatbot mengarahkan pengguna untuk melakukan manajemen tugas (seperti menandai selesai atau mengatur pengingat). Integrasi dengan Firebase memastikan sinkronisasi data antara chatbot dan backend.

---

## 🚧 **3. Cara Kerja Modul**

### Alur Kerja

1. **Pengguna Bertanya atau Memberi Perintah**:
   - Chatbot menangkap input dari pengguna, seperti perintah untuk melihat status tugas atau menandai tugas selesai.
2. **Webhook untuk Pemrosesan Lebih Lanjut**:
   - Jika tugas melibatkan update data (seperti menandai tugas selesai), webhook akan mengirim perintah ke server Node.js untuk berinteraksi dengan Firebase.
3. **Sinkronisasi Firebase**:
   - Chatbot menggunakan Firebase untuk memeriksa status tugas atau memperbarui informasi tugas sesuai permintaan pengguna.

### Diagram Alur

1. **User Input** ➔ **Chatbot** ➔ **Webhook/API ke Server** ➔ **Firebase** ➔ **Chatbot Response ke User**
2. **User Menyelesaikan Tugas** ➔ **Webhook/API menandai tugas selesai** ➔ **Update Firebase** ➔ **Confirm ke Chatbot**

---

## 🛠️ **4. Tugas dan Fungsi Utama dalam Pengembangan**

### A. **Setup Proyek**

- **Tugas**: Membuat struktur proyek untuk server Node.js dan menginstal dependensi.
- **Langkah**:
  1. Inisialisasi proyek Node.js:
     ```bash
     npm init -y
     npm install express firebase-admin dialogflow jsonwebtoken dotenv
     ```
  2. Konfigurasi `.env` untuk Firebase dan JWT.

### B. **Integrasi Chatbot dengan Dialogflow**

- **Tugas**: Mengonfigurasi chatbot di Dialogflow untuk menangani percakapan dengan pengguna.
- **Langkah**:
  1. Buat project baru di Dialogflow dan buat intent sesuai fitur yang ingin didukung (seperti "Cek Status Tugas" atau "Tandai Selesai").
  2. Konfigurasi intent dengan respons default dan webhook yang mengarahkan ke server Node.js.

### C. **Membangun Webhook untuk Dialogflow**

- **Tugas**: Membangun endpoint untuk menerima permintaan dari Dialogflow dan memberikan respons.
- **Langkah**:

  1. Buat route `dialogflowWebhook` pada server yang akan menerima data intent dari Dialogflow.
  2. Endpoint ini akan memproses intent, seperti memeriksa status tugas atau menandai tugas selesai, lalu mengirim respons balik ke Dialogflow.

  - *Contoh Endpoint*:
    ```javascript
    app.post('/dialogflow-webhook', (req, res) => {
        const intent = req.body.queryResult.intent.displayName;
        if (intent === 'Tandai Selesai') {
            // Logic untuk tandai tugas selesai
        } else if (intent === 'Cek Status Tugas') {
            // Logic untuk cek status tugas
        }
        res.json({ fulfillmentText: "Respons Chatbot" });
    });
    ```

### D. **Sinkronisasi dengan Firebase untuk Status Tugas**

- **Tugas**: Mengambil data dari Firebase terkait status tugas dan memperbaruinya saat pengguna memberikan perintah.
- **Langkah**:

  1. Gunakan Firebase SDK untuk menghubungkan chatbot ke database dan melakukan operasi CRUD pada tugas.
  2. Buat fungsi `getTaskStatus` untuk mengecek status tugas tertentu, serta fungsi `markTaskAsCompleted` untuk memperbarui status.

  - *Contoh Kode*:
    ```javascript
    const getTaskStatus = async (taskId) => {
        const taskRef = db.collection('tasks').doc(taskId);
        const doc = await taskRef.get();
        return doc.exists ? doc.data() : null;
    };
    const markTaskAsCompleted = async (taskId) => {
        await db.collection('tasks').doc(taskId).update({ status: 'completed' });
    };
    ```

### E. **Mengatur Pengingat Tugas dengan Dialogflow**

- **Tugas**: Mengatur pengingat otomatis untuk tugas menggunakan Firebase dan mengirim respons notifikasi ke pengguna.
- **Langkah**:

  1. Dialogflow dapat mengirimkan perintah pengingat menggunakan intent "Pengingat Tugas".
  2. Gunakan Firebase Cloud Functions atau scheduler pada server Node.js untuk mengatur pengingat.

  - *Contoh Cloud Function*:
    ```javascript
    exports.sendReminder = functions.firestore.document('tasks/{taskId}')
        .onUpdate(async (change, context) => {
            // Logic to send reminder based on dueDate
        });
    ```

---

## 🔗 **5. Struktur Proyek**

```
ChatTasker-Chatbot/
├── src/
│   ├── controllers/
│   │   ├── dialogflowController.js
│   ├── config/
│   │   ├── firebaseConfig.js
│   ├── routes/
│   │   ├── dialogflowRoutes.js
│   ├── index.js
├── .env
├── package.json
└── README.md
```

---

## 🔑 **6. Konfigurasi dan Variabel Lingkungan**

### `.env`

```
FIREBASE_PROJECT_ID=<YOUR_PROJECT_ID>
FIREBASE_CLIENT_EMAIL=<YOUR_CLIENT_EMAIL>
FIREBASE_PRIVATE_KEY=<YOUR_PRIVATE_KEY>
```

### `firebaseConfig.js`

```javascript
const admin = require('firebase-admin');
const serviceAccount = require('path/to/firebase-adminsdk.json');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

const db = admin.firestore();
module.exports = { db };
```

---

## 📌 **7. API Endpoint dan Dokumentasi**

| HTTP Method | Endpoint                | Description                         |
| ----------- | ----------------------- | ----------------------------------- |
| POST        | `/dialogflow-webhook` | Menerima permintaan dari Dialogflow |
| GET         | `/tasks/:id/status`   | Mendapatkan status tugas tertentu   |
| PUT         | `/tasks/:id/complete` | Menandai tugas sebagai selesai      |

---

## 🔒 **8. Keamanan dan Autentikasi**

- Gunakan **JWT** untuk endpoint webhook jika ada fitur login.
- Middleware `authMiddleware.js` dapat ditambahkan untuk memeriksa token pada endpoint webhook.

---

## 📝 **9. Panduan Pengembangan**

1. **Clone Repositori**:

   ```bash
   git clone https://github.com/yourusername/ChatTasker-Chatbot.git
   cd ChatTasker-Chatbot
   ```
2. **Instalasi Dependensi**:

   ```bash
   npm install
   ```
3. **Jalankan Aplikasi**:

   ```bash
   npm start
   ```
4. **Testing API**: Gunakan Postman untuk menguji webhook dan sinkronisasi dengan Firebase.

---

## ✅ **10. Testing dan Pengujian**

- **Unit Testing**: Buat unit test pada `dialogflowController.js` menggunakan Jest atau Mocha.
- **Integration Testing**: Uji integrasi antara webhook dan Firebase.
- **E2E Testing**: Lakukan uji end-to-end untuk memastikan respons chatbot sesuai dengan intent pengguna dan data di Firebase.

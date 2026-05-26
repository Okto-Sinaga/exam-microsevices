# Panduan Pengaturan Sistem Terdistribusi: Laptop 1 & Laptop 3

Dokumen ini berisi panduan teknis langkah demi langkah untuk membagi sistem menjadi dua antarmuka (web) terpisah yang terhubung pada sistem database dan backend terpusat yang sama di **Laptop 1**.

```mermaid
graph TD
    subgraph Laptop 3 (Exam Portal)
        FE3[Frontend Next.js: Ujian & Beranda]
    end

    subgraph Laptop 1 (Core Portal & Central Backend)
        FE1[Frontend Next.js: Katalog, Kelas & Pencapaian]
        GW[Nginx API Gateway - Port 8080]
        
        subgraph Backend Services
            AS[Auth Service - Port 3001]
            CS[Course Service - Port 3002/3003]
            ES[Exam Service - Port 8082]
        end
        
        subgraph Databases
            MDB[(MongoDB - lms)]
            PGDB[(PostgreSQL - exam_db & authdb)]
        end
    end

    FE3 -- HTTP/API Request --> GW
    FE1 -- HTTP/API Request --> GW
    
    GW --> AS
    GW --> CS
    GW --> ES
    
    AS --> PGDB
    CS --> MDB
    ES --> PGDB
```

---

## 💻 Bagian 1: Pengaturan di Laptop 1 (Host Utama & Database)

Laptop 1 bertindak sebagai **pusat data dan gerbang API utama** untuk seluruh kelompok.

### 1. Dapatkan Alamat IP Lokal Laptop 1
Pastikan Laptop 1 dan Laptop 3 terhubung ke jaringan Wi-Fi yang sama.
- Alamat IP Wifi Laptop 1 Anda saat ini adalah: **`172.30.59.189`**

### 2. Modifikasi Frontend di Laptop 1 (Fokus ke Courses & Achievements)
Di folder proyek frontend Laptop 1 Anda:
1. Buka file komponen navigasi/nav bar (biasanya `Navbar.tsx`, `Sidebar.tsx`, atau di folder `components/`).
2. Cari link menu navigasi kearah halaman **Ujian** (`/exams` atau `/quizzes`).
3. Hapus atau komentari kode link tersebut agar tab **Ujian** hilang dari navigasi Laptop 1.
4. Pastikan file `.env` frontend Laptop 1 mengarah ke API Gateway lokal:
   ```env
   NEXT_PUBLIC_API_URL=http://localhost:8080
   ```
5. Jalankan frontend Next.js Laptop 1 (`npm run dev`). Web ini sekarang hanya fokus menampilkan Katalog Kursus, Pembelajaran Saya, dan Pencapaian.

---

## 💻 Bagian 2: Pengaturan di Laptop 3 (Exam Web Portal)

Laptop 3 akan menjalankan aplikasi frontend Next.js yang **khusus menangani modul Ujian**.

### 1. Salin Kode Frontend ke Laptop 3
- Copy folder proyek frontend Next.js dari Laptop 1 ke Laptop 3 (via Flashdisk, Git, atau LAN share).

### 2. Konfigurasi Alamat API Sentral di Laptop 3
Agar Laptop 3 bisa menembak database di Laptop 1:
1. Di Laptop 3, buka file konfigurasi environment `.env` atau `.env.local` di folder root Next.js.
2. Ubah URL API agar mengarah ke IP Laptop 1 di port `8080`:
   ```env
   NEXT_PUBLIC_API_URL=http://172.30.59.189:8080
   ```
   *(Ganti `172.30.59.189` dengan IP Wifi Laptop 1 jika di kemudian hari IP Laptop 1 berubah).*

### 3. Modifikasi Navigasi Frontend di Laptop 3 (Hanya Menampilkan Menu Ujian)
Buka kode komponen navigasi di Laptop 3:
1. Hapus atau komentari menu navigasi untuk:
   - **Katalog Kursus** (`/courses`)
   - **Pembelajaran Saya** (`/my-learning`)
2. Sisakan menu navigasi untuk:
   - **Beranda** (`/dashboard`)
   - **Ujian** (`/exams`)
   - **Pencapaian** (`/achievements`)
3. Jalankan server Next.js di Laptop 3:
   ```bash
   npm run dev
   ```

---

## 🧪 Skenario Demo Pengujian Terdistribusi (Dua Web Satu Sistem)

Berikut adalah cara kelompok Anda mendemokan integrasi ini kepada dosen/asisten praktikum:

### **Skenario A: Registrasi & Login Sentral**
1. Buka browser di **Laptop 3** (`http://localhost:3000`) dan lakukan registrasi akun siswa baru bernama **"Budi Terdistribusi"** (`budi@edu.ai`).
2. Data akun Budi akan terkirim ke Laptop 1 dan disimpan ke PostgreSQL `authdb` Laptop 1.
3. Sekarang, login dengan akun `budi@edu.ai` di **Laptop 1** (`http://localhost:3000`) dan **Laptop 3** (`http://localhost:3000`). Kedua laptop berhasil login karena menggunakan auth service terpusat yang sama.

### **Skenario B: Alur Mengikuti Kelas & Ujian Terintegrasi**
1. **Di Laptop 1 (Katalog/Kelas):** 
   - Budi membuka tab Katalog, memilih kelas **Blockchain**, lalu klik **Tambahkan ke Pembelajaran**.
   - Kelas Blockchain masuk ke MongoDB Laptop 1.
2. **Di Laptop 3 (Ujian):**
   - Budi membuka tab **Ujian** (yang berjalan di Laptop 3).
   - Di sini, Budi memilih **Ujian Dasar Blockchain** dan mengerjakannya hingga selesai, lalu klik **Kumpulkan**.
   - Riwayat ujian terkirim ke `exam-service` di Laptop 1 dan disimpan ke PostgreSQL `exam_db` Laptop 1.
3. **Di Laptop 1 & Laptop 3 (Pencapaian):**
   - Budi membuka tab **Pencapaian** di Laptop 1 maupun di Laptop 3.
   - **Hasil:** Kedua laptop akan menampilkan riwayat nilai yang sama secara real-time karena keduanya membaca tabel `exam_attempts` yang sama di database PostgreSQL Laptop 1.

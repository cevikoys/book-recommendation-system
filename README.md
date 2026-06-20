# Sistem Rekomendasi Buku Berbasis Web Menggunakan Metode Content-Based Filtering dan Google Books API

Proyek ini merupakan implementasi purwapura (*prototype*) Sistem Rekomendasi Buku yang mengintegrasikan sumber data dinamis melalui antarmuka pemrograman aplikasi (API). Sistem ini dirancang untuk menyelesaikan permasalahan pencarian literatur yang relevan dengan menerapkan algoritma pencocokan teks (*text matching*) mandiri di sisi klien (*client-side*).

Pengembangan proyek ini ditujukan sebagai pemenuhan Tugas Akhir / Ujian Akhir Semester (UAS) pada mata kuliah Sistem Rekomendasi.

---

## 1. Metodologi & Arsitektur Sistem

Sistem rekomendasi ini bekerja secara *asynchronous* dengan memanfaatkan arsitektur hibrida antara penyediaan data eksternal dan pemrosesan logika lokal:

### A. Akuisisi Data Dinamis (Data Acquisition)
Sistem melakukan panggilan *query* secara *real-time* ke **Google Books API** menggunakan fungsi *asynchronous* (`fetch` dengan mekanisme `async/await`). Parameter pencarian dibatasi maksimal 20 item per request untuk menjaga efisiensi performa transfer data. Metadata yang diambil meliputi komponen: judul, kategori (genre), deskripsi teks, tanggal publikasi, tautan gambar sampul, dan rating rata-rata (*average rating*).

### B. Algoritma Pencocokan Teks (Content-Based Filtering)
Sistem menerapkan pendekatan berbasis konten (*Content-Based Filtering*) sederhana melalui tokenisasi kata kunci kueri (`queryWords`) terhadap gabungan korpus teks objek buku (`title`, `genre`, dan `description`). 

Pembobotan matriks kesamaan (*similarity score*) dihitung secara kumulatif dengan aturan metrik berikut:
* Setiap kemunculan kata kunci yang cocok di dalam korpus teks mendapatkan bobot awal $+1$.
* Jika kata kunci pencarian cocok secara spesifik pada komponen judul (`title`), sistem memberikan bobot tambahan $+2$ untuk memprioritaskan relevansi utama.

Buku yang memiliki bobot akhir $>0$ akan difilter, kemudian diurutkan secara menurun (*descending ranking*) berdasarkan skor tertinggi sebelum dirender ke antarmuka pengguna.

---

## 2. Struktur Berkas & Komponen

* **`home.html`**  
  Dokumen inti yang mengintegrasikan kerangka kerja antarmuka (*User Interface*) dan seluruh blok skrip logika (fungsi inferensi rekomendasi, fungsi kalkulasi *star rating*, dan penanganan *payload* API).
* **`style.css`**  
  Modul gaya eksternal yang mengatur visualisasi layout. Menggunakan implementasi Flexbox untuk penataan grid kartu buku (*book cards*) yang responsif, manajemen interaksi *pseudo-class* (`:hover`), dan pengaturan hierarki visual.
* **`Background_Buku.jpg`**  
  Aset grafis lokal yang berfungsi sebagai latar belakang estetika halaman guna meningkatkan *user experience* (UX).
* **`Link Youtube_Book Recommendation.pdf`**  
  Dokumen penunjang akademis yang memuat tautan dokumentasi video. Video tersebut menjabarkan demonstrasi teknis jalannya sistem, pengujian fungsionalitas, serta pemaparan alur logika algoritma.

---

## 3. Implementasi Kode Utama

Evaluasi kedekatan teks dijalankan melalui fungsi kalkulasi skor berikut:

```javascript
function calculateSimilarity(query, book) {
    query = query.toLowerCase();
    const bookText = `${book.title} ${book.genre} ${book.description}`.toLowerCase();
    const queryWords = query.split(/\s+/);
    let score = 0;

    queryWords.forEach(word => {
        if (bookText.includes(word)) {
            score += 1;
        }
    });

    if (book.title.toLowerCase().includes(query)) {
        score += 2;
    }
    return score;
}

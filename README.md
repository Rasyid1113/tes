Tapak Hayati — Pemantauan Keanekaragaman Hayati
Website satu-halaman untuk mencatat pengamatan spesies (nama + foto opsional), menandainya di peta, dan melihat kawasan mana yang tergolong kaya, sedang, atau minim keanekaragaman hayati berdasarkan data yang terkumpul.
Tidak ada proses build — cukup file statis (`index.html`), jadi bisa langsung dibuka di peramban atau di-hosting di GitHub Pages.
Fitur
Input pengamatan: nama spesies, catatan bebas, foto (dikompresi otomatis di sisi klien sebelum disimpan), dan lokasi (klik di peta atau pakai GPS perangkat).
Peta interaktif (Leaflet + OpenStreetMap):
Lapisan titik pengamatan — tiap pengamatan sebagai titik berwarna.
Lapisan kawasan (agregat) — data dikelompokkan ke dalam grid, tiap sel diwarnai sesuai jumlah spesies unik di dalamnya:
Tinggi (hijau) — 5+ spesies unik
Sedang (kuning tanah) — 2–4 spesies unik
Rendah (merah bata) — 1 spesies / data masih sedikit
Ukuran grid kawasan bisa diubah (Detail ~2 km, Sedang ~5 km, Luas ~11 km).
Ringkasan kawasan: tabel yang mengurutkan kawasan dari yang paling kaya spesies, bisa diklik untuk terbang ke lokasinya di peta.
Statistik ringkas di bagian atas: total spesies unik, total pengamatan, jumlah kawasan berkategori "tinggi".
Menjalankan secara lokal
Cukup buka `index.html` langsung di peramban, atau jalankan server statis sederhana, misalnya:
```bash
npx serve .
# atau
python3 -m http.server 8000
```
Deploy ke GitHub Pages
Push folder ini ke sebuah repository GitHub.
Buka Settings → Pages pada repo tersebut.
Pilih source Deploy from a branch, branch `main`, folder `/root`.
Simpan — situs akan tersedia di `https://<username>.github.io/<nama-repo>/` setelah beberapa menit.
Catatan teknis & keterbatasan
Penyimpanan data: memakai `localStorage` di peramban pengguna. Artinya data bersifat lokal per perangkat/peramban, tidak otomatis tersinkron antar pengguna atau perangkat. Ini cukup untuk uji coba, riset kecil, atau penggunaan satu tim di satu perangkat bersama.
Untuk pemantauan kolaboratif sungguhan (banyak orang input dari lokasi berbeda), sambungkan form ini ke backend + database (mis. Supabase, Firebase, atau API sendiri) — bagian yang perlu diganti hanya fungsi `loadObservations()` dan `saveObservations()` di dalam `index.html`.
Definisi "kawasan": didekati dengan membagi peta menjadi grid berbasis derajat lintang/bujur (bukan proyeksi yang mengoreksi kelengkungan bumi), jadi ukuran kawasan sedikit berbeda tergantung garis lintang. Cukup akurat untuk eksplorasi visual, tapi bukan analisis spasial presisi tinggi.
Ambang kategori (tinggi/sedang/rendah) memakai angka tetap (5+/2–4/1) yang bisa diubah langsung di fungsi `classify()` pada kode, disesuaikan dengan konteks studi (mis. indeks Shannon-Wiener atau Simpson bila ingin lebih ilmiah).
Foto dikompresi ke lebar maksimum 480px sebelum disimpan agar tidak membebani `localStorage` (yang punya batas kapasitas, umumnya sekitar 5–10 MB per situs).

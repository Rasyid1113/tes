Tapak Hayati — Pemantauan Keanekaragaman Hayati
Website satu-halaman untuk mencatat pengamatan spesies (nama + foto opsional), menandainya di peta, dan melihat kawasan mana yang tergolong kaya, sedang, atau minim keanekaragaman hayati berdasarkan data yang terkumpul.
Tidak ada proses build — cukup file statis (`index.html`), jadi bisa langsung dibuka di peramban atau di-hosting di GitHub Pages.
Fitur
Input pengamatan: nama spesies, catatan bebas, foto (dikompresi otomatis di sisi klien sebelum disimpan), dan lokasi (klik di peta atau pakai GPS perangkat).
Peta interaktif (Leaflet + OpenStreetMap) dengan lapisan titik pengamatan dan lapisan kawasan (agregat) yang diwarnai menurut jumlah spesies unik:
Tinggi (hijau) — 5+ spesies unik
Sedang (kuning tanah) — 2–4 spesies unik
Rendah (merah bata) — 1 spesies / data masih sedikit
Ukuran grid kawasan bisa diubah (Detail ~2 km, Sedang ~5 km, Luas ~11 km).
Ringkasan kawasan: tabel yang mengurutkan kawasan dari yang paling kaya spesies, bisa diklik untuk terbang ke lokasinya di peta.
Statistik ringkas: total spesies unik, total pengamatan, jumlah kawasan berkategori "tinggi".
Dua mode penyimpanan:
Mode lokal (default) — data disimpan di `localStorage` peramban, hanya terlihat di perangkat itu sendiri.
Mode bersama — data disimpan di database Supabase, langsung terlihat oleh siapa pun yang membuka situs, dan pembaruan dari orang lain otomatis muncul (realtime) tanpa perlu refresh.
Menjalankan secara lokal
Cukup buka `index.html` langsung di peramban, atau jalankan server statis sederhana:
```bash
npx serve .
# atau
python3 -m http.server 8000
```
Deploy ke GitHub Pages
Push folder ini ke sebuah repository GitHub.
Buka Settings → Pages pada repo tersebut.
Pilih source Deploy from a branch, branch `main`, folder `/root`.
Situs akan tersedia di `https://<username>.github.io/<nama-repo>/` setelah beberapa menit.
Mengaktifkan mode bersama (Supabase) — supaya data terlihat orang lain
Tanpa langkah ini, situs tetap berjalan normal tapi setiap orang punya datanya sendiri-sendiri di perangkat masing-masing. Ikuti langkah ini agar semua orang melihat data yang sama:
Buat project Supabase gratis di supabase.com.
Buka SQL Editor di dashboard project, lalu jalankan:
```sql
   create table observations (
     id text primary key,
     species text not null,
     note text,
     photo text,
     lat double precision not null,
     lng double precision not null,
     created_at timestamptz not null default now()
   );

   alter table observations enable row level security;

   -- kebijakan publik: siapa saja bisa membaca, menambah, dan menghapus.
   -- cocok untuk proyek pemantauan komunitas yang terbuka.
   -- jika ingin lebih terkontrol, hapus policy "delete" di bawah dan
   -- kelola penghapusan lewat dashboard Supabase saja.
   create policy "Public read" on observations for select using (true);
   create policy "Public insert" on observations for insert with check (true);
   create policy "Public delete" on observations for delete using (true);

   -- aktifkan realtime supaya perubahan langsung muncul di semua pengunjung
   alter publication supabase_realtime add table observations;
   ```
Buka Settings → API, salin Project URL dan anon public key.
Di `index.html`, cari bagian ini di dekat awal tag `<script>` dan isi dengan nilai dari langkah 3:
```js
   var SUPABASE_URL = 'YOUR-PROJECT-REF.supabase.co';
   var SUPABASE_ANON_KEY = 'YOUR-ANON-PUBLIC-KEY';
   ```
Simpan, lalu deploy ulang (push ke GitHub). Badge di pojok kiri atas situs akan berubah menjadi "Mode bersama".
Penting soal keamanan: karena `anon key` tertanam di kode sisi klien (bisa dilihat siapa saja lewat "view source"), kebijakan di atas membuat tabel benar-benar terbuka — siapa pun bisa menambah maupun menghapus data siapa saja. Ini wajar untuk proyek pemantauan komunitas/citizen-science yang memang terbuka, tapi juga berarti rawan spam atau penghapusan iseng. Untuk kontrol lebih ketat di kemudian hari:
Hapus kebijakan `delete` publik dan kelola penghapusan hanya lewat dashboard Supabase (sebagai admin).
Tambahkan autentikasi pengguna (Supabase Auth) sehingga hanya pengguna yang login yang bisa menambah/menghapus data miliknya sendiri.
Catatan teknis & keterbatasan
Definisi "kawasan": didekati dengan membagi peta menjadi grid berbasis derajat lintang/bujur (bukan proyeksi yang mengoreksi kelengkungan bumi), jadi ukuran kawasan sedikit berbeda tergantung garis lintang. Cukup akurat untuk eksplorasi visual, tapi bukan analisis spasial presisi tinggi.
Ambang kategori (tinggi/sedang/rendah) memakai angka tetap (5+/2–4/1) yang bisa diubah langsung di fungsi `classify()` pada kode, disesuaikan dengan konteks studi (mis. indeks Shannon-Wiener atau Simpson bila ingin lebih ilmiah).
Foto dikompresi ke lebar maksimum 420px sebelum disimpan sebagai teks base64. Ini menyederhanakan implementasi, tapi untuk banyak foto beresolusi lebih tinggi, pertimbangkan memakai Supabase Storage (bucket file) alih-alih menyimpan foto sebagai teks di dalam tabel.

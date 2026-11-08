# Bagian B - Membangun API Microservice KatalogKu di STB

Panduan kerja ini mengikuti urutan B1-B16 pada Guide Bagian B, yang sama dengan slide 35-50 pada Full PPT Guide TST. Pembuka monolith/microservice ada pada slide 34. Berkas konfigurasi berasal dari ZIP referensi yang diberikan pengguna.

**Status:** konfigurasi, pembuatan koleksi, pengisian lima produk, REST Client, dan Postman telah dilaksanakan. Hasil Runner: 9 Passed, 0 Failed, 0 Errors; respons registrasi 400 sudah dipastikan karena email duplikat. Berkas pb_schema telah diganti dengan hasil Export aktual. Perintah dan contoh respons berikut tetap menjadi panduan penggunaan.

## Persiapan server

Gunakan server yang sudah disiapkan pada Bagian A. Ganti `IP_SERVER` pada contoh URL dengan alamat server Anda. Periksa Docker Compose, arsitektur CPU, port 8080/8091/8092, dan ruang penyimpanan sebelum build. Konfigurasi mendukung ARM64 dan amd64 sesuai modul. Simpan catatan kondisi server dan bukti terminal dalam laporan pribadi.

## Pembuka - Monolith vs Microservice (slide 34)

Monolith menggabungkan fitur dalam satu aplikasi besar. Microservice memisahkan tanggung jawab ke beberapa service yang berkomunikasi lewat HTTP/REST dengan JSON. KatalogKu menggunakan pemisahan ini untuk mempelajari integrasi sistem: gateway menerima request, service menangani tanggung jawabnya, dan setiap service menyimpan datanya sendiri.

## B1 - PocketBase: Backend Open Source Tanpa Coding (slide 35)

PocketBase versi **0.40.5** mengikuti versi yang ditetapkan panduan. PocketBase menyediakan database SQLite, dashboard admin, REST API otomatis, autentikasi, dan API Rules. Pembentukan backend dilakukan melalui konfigurasi dan dashboard.

PocketBase menyimpan data masing-masing service di SQLite. MySQL dan phpMyAdmin dari Bagian A tetap merupakan hasil setup server, sedangkan data produk dan akun Bagian B berada di volume PocketBase.

## B2 - Rancangan Service KatalogKu (slide 36)

| Service | Isi | Akses dari laptop | Data | Prefix gateway |
|---|---|---|---|---|
| gateway | Nginx, meneruskan request | :8080 | - | /api/... |
| product-service | PocketBase #1, products | :8091, dashboard | product_data / SQLite sendiri | /api/products |
| account-service | PocketBase #2, users dan favorites | :8092, dashboard | account_data / SQLite sendiri | /api/auth, /api/favorites |

Tiap PocketBase mendengarkan port 8090 di jaringan internal Docker. Port host 8091 dan 8092 dipetakan ke port internal tersebut. `favorites.product_id` merupakan teks berisi ID produk dari product-service. Detail produk diminta melalui API product-service oleh aplikasi.

## B3 - Arsitektur dan Alur Request (slide 37)

Client memanggil `http://IP_SERVER:8080/api`. Gateway meneruskan request produk ke product-service serta request akun dan favorit ke account-service.

Urutan alur favorit:

1. Client login melalui `/api/auth/auth-with-password` dan menerima token serta `record.id`.
2. Client mengirim POST `/api/favorites`, header `Authorization: Bearer <token>`, dan body `user` serta `product_id`.
3. Gateway meneruskan request ke account-service.
4. API Rules memeriksa kepemilikan user.
5. Untuk menampilkan detail favorit, aplikasi memanggil `/api/products/:id` menggunakan `product_id`.

## B4 - Struktur Folder Proyek (slide 38)

```text
katalogku-pocketbase/
  docker-compose.yml
  pocketbase/Dockerfile
  gateway/nginx.conf
  pb_schema/product_collections.json
  pb_schema/account_collections.json
  seed.http
  api.http
  postman/
  .env.example
  .gitignore
  README.md
  PANDUAN_B_STB.md
  CHECKLIST_B.md
```

Dua service menggunakan Dockerfile PocketBase yang sama dan volume berbeda. `api.http`, `seed.http`, dan environment Postman ServerLAN menggunakan placeholder IP_SERVER yang harus diganti pada salinan lokal. Password pada template uji sengaja kosong dan harus diisi pada salinan lokal.

## B5 - Dockerfile PocketBase (slide 39)

Periksa `pocketbase/Dockerfile`:

- Image dasar `alpine:3.22`.
- `ARG PB_VERSION=0.40.5`.
- Docker mengisi `TARGETARCH`; STB menggunakan `arm64`.
- Binary berasal dari rilis PocketBase untuk Linux dengan arsitektur tersebut.
- `serve --http=0.0.0.0:8090 --dir=/pb_data` menyimpan database dan file pada volume `/pb_data`.

Dockerfile dipertahankan dari ZIP. Tidak ada pembaruan versi otomatis dalam pengerjaan ini.

## B6 - docker-compose.yml (slide 40)

Periksa pemetaan berikut pada `docker-compose.yml`:

| Service | Build / image | Volume | Port host:container |
|---|---|---|---|
| product-service | build ./pocketbase | product_data:/pb_data | 8091:8090 |
| account-service | build ./pocketbase | account_data:/pb_data | 8092:8090 |
| gateway | nginx:1.28-alpine | nginx.conf ke default.conf, read-only | 8080:80 |

Ketiganya menggunakan `restart: unless-stopped`. Gateway memiliki `depends_on` terhadap kedua PocketBase. Service `tunnel` pada ZIP memakai profile `tunnel` dan tidak ikut berjalan pada perintah Compose biasa. Cloudflare dibahas di Bagian C.

## B7 - API Gateway: Memetakan URL ke PocketBase (slide 41)

Konfigurasi `gateway/nginx.conf` mengikuti ZIP:

| URL client | Tujuan internal |
|---|---|
| /api/health | respons JSON dari gateway |
| /api/products | product-service:8090/api/collections/products/records |
| /api/products/:id | product-service:8090/api/collections/products/records/:id |
| /api/auth/records | account-service:8090/api/collections/users/records |
| /api/auth/auth-with-password | account-service:8090/api/collections/users/auth-with-password |
| /api/favorites | account-service:8090/api/collections/favorites/records |
| /api/favorites/:id | account-service:8090/api/collections/favorites/records/:id |

Nginx meneruskan Host, X-Real-IP, dan X-Forwarded-For. Batas body adalah 5 MB. Gateway memberi satu alamat API untuk client.

## B8 - Jalankan dan Buat Akun Superuser (slide 42)

### 1. Ambil proyek ke STB

Semua perintah pada blok ini dijalankan di **terminal SSH STB**. Jika folder tujuan sudah ada, periksa isinya terlebih dahulu agar tidak mencampur pekerjaan lama.

```bash
mkdir -p /www/apps
cd /www/apps
git clone https://github.com/arkandhaf/TST-69.git katalogku-pocketbase
cd /www/apps/katalogku-pocketbase
cp -n .env.example .env
docker compose config --quiet
```

`.env` berisi nilai tunnel kosong dan diabaikan Git. Bagian B tidak membutuhkan token Cloudflare.

### 2. Jalankan service

Setelah memastikan lokasi dan ruang penyimpanan Docker serta ketersediaan port, jalankan di terminal SSH server:

```bash
docker compose up -d --build
docker compose ps
docker compose exec gateway nginx -t
curl -i http://localhost:8080/api/health
```

Hasil yang diharapkan: tiga service Bagian B berstatus Up, konfigurasi Nginx valid, serta health HTTP 200 dengan body `{"gateway":"ok"}`. Health gateway saja belum membuktikan koleksi PocketBase sudah siap. Jika gagal, periksa log:

```bash
docker compose logs --tail=80 product-service account-service gateway
```

### 3. Buat superuser di kedua service

Email admin mengikuti contoh modul `admin@katalogku.id`. Pilih password baru minimum 10 karakter. Password diketik tersembunyi dan tidak perlu dikirim ke chat.

Jalankan dalam shell Bash pada sesi SSH:

```bash
read -r -s -p 'Password admin baru (minimum 10 karakter): ' PB_ADMIN_PASSWORD
printf '\n'
if [ "${#PB_ADMIN_PASSWORD}" -lt 10 ]; then
  printf 'Password terlalu pendek. Ulangi tahap ini.\n'
else
  docker compose exec -T product-service pocketbase superuser upsert admin@katalogku.id "$PB_ADMIN_PASSWORD" --dir=/pb_data
  docker compose exec -T account-service pocketbase superuser upsert admin@katalogku.id "$PB_ADMIN_PASSWORD" --dir=/pb_data
fi
unset PB_ADMIN_PASSWORD
```

Pastikan kedua perintah melaporkan keberhasilan. Buka dashboard **dari browser laptop**:

- Product: http://IP_SERVER:8091/_/
- Account: http://IP_SERVER:8092/_/

Login dengan email dan password yang baru dibuat. Akun ini adalah admin dashboard, berbeda dari akun pengguna aplikasi pada koleksi `users`.

## B9 - Buat Koleksi products di product-service (slide 43)

Slide B9 menjelaskan definisi koleksi. B10 menyediakan jalur cepat melalui import. Pada pengerjaan ini gunakan import B10 untuk membentuk koleksi, kemudian cocokkan seluruh field dan rules B9 di dashboard. Dengan demikian definisi B9 tetap diperiksa lengkap dan data tidak dimasukkan dua kali.

| Field | Tipe | Pengaturan yang diperiksa |
|---|---|---|
| title | Plain text | Nonempty / required |
| description | Plain text | opsional |
| category | Plain text | Nonempty / required |
| price | Number | Nonempty / required, minimum 0 |
| rating | Number | minimum 0, maksimum 5 |
| stock | Number | No decimals / integer |
| thumbnail | URL | tautan gambar |

API Rules products:

| Rule | Nilai |
|---|---|
| List | terbuka, kosong |
| View | terbuka, kosong |
| Create | terkunci / null |
| Update | terkunci / null |
| Delete | terkunci / null |

Terkunci berarti hanya superuser. Terbuka dan kosong berarti publik. Terbuka dengan ekspresi berarti akses mengikuti ekspresi tersebut. Lima produk contoh diisi satu kali pada B10.

Jika memilih jalur manual dari modul, gunakan New collection > Base > products, isi field dan rules di atas, lalu Create. Untuk menghindari benturan ID, jangan mengimpor lagi skema referensi ke koleksi manual yang ID-nya berbeda. Ekspor hasil manual sebagai skema aktual untuk repository.

## B10 - Cara Cepat: Import Skema dan Data Contoh (slide 44)

### 1. Siapkan salinan proyek di laptop

Browser memilih file JSON dari laptop. Jika belum ada salinan lokal, jalankan di **PowerShell laptop**, pada folder kerja yang Anda pilih:

```powershell
git clone https://github.com/arkandhaf/TST-69.git katalogku-pocketbase
Set-Location -LiteralPath .\katalogku-pocketbase
```

Gunakan salinan lokal yang telah disiapkan jika sudah tersedia, sehingga tidak perlu clone kedua kalinya.

### 2. Import kedua skema

1. Dashboard product-service :8091 > Settings > Import collections > Load from JSON file.
2. Pilih `pb_schema/product_collections.json` dari laptop.
3. Aktifkan **Merge with the existing collections** sebelum Review. Pada PocketBase 0.40.5 opsi ini awalnya tidak aktif, sehingga skema parsial bisa menampilkan koleksi internal sebagai Deleted.
4. Klik Review. Untuk import produk pada instance baru, perubahan yang diharapkan hanya **Added products**. Pastikan tidak ada penghapusan koleksi atau field yang sudah ada, terutama _superusers, users, dan koleksi internal. Jika muncul dialog penghapusan, pilih No, tutup Review, dan periksa kembali opsi Merge.
5. Setelah preview sesuai, klik Confirm and import.
6. Dashboard account-service :8092 > Settings > Import collections > Load from JSON file. Pilih `pb_schema/account_collections.json` dan aktifkan **Merge with the existing collections** juga.
7. Jika muncul peringatan bahwa nama koleksi/field sama tetapi ID berbeda, gunakan **Replace with original IDs**. PocketBase menyesuaikan ID serta referensi relation dalam data import dengan koleksi yang sudah ada. Review kembali; jangan menyetujui penghapusan koleksi internal.
8. Perubahan account-service yang diharapkan adalah penambahan favorites serta penyesuaian users sesuai skema. Setelah preview sesuai, Confirm and import.
9. Cocokkan products dengan tabel B9 dan users/favorites dengan tabel B11.

Import dilakukan pada service yang sesuai. Skema produk dan akun berada di database terpisah.

### 3. Isi lima produk contoh

VS Code dengan REST Client digunakan pada B13-B14. Untuk tahap pengisian ini, buka proyek lokal di VS Code dan siapkan file lokal di **PowerShell laptop**:

```powershell
Copy-Item -LiteralPath .\seed.http -Destination .\seed.local.http
```

Di `seed.local.http`, isi `@adminPassword =` dengan password superuser product-service dari B8. Ganti `IP_SERVER` pada `@pbUrl = http://IP_SERVER:8091` dengan alamat server Anda.

1. Send Request pada **0. Login superuser**. Harus HTTP 200 dan ada token.
2. Send Request satu per satu untuk produk 1-5. Masing-masing harus HTTP 200 dan memiliki ID.
3. Periksa dashboard products: lima record tersedia.

Data contoh persis dari ZIP:

| Produk | Category | Price | Rating | Stock |
|---|---|---:|---:|---:|
| Kemeja Batik Parang | fashion | 25 | 4.6 | 40 |
| Kopi Gayo 250 gr | groceries | 9 | 4.8 | 120 |
| Teh Hijau Organik | groceries | 6.5 | 4.3 | 80 |
| Earphone Bluetooth | electronics | 19 | 4.2 | 60 |
| Power Bank 10000 mAh | electronics | 15 | 4.5 | 50 |

Nilai price dipertahankan dari referensi tanpa menambahkan asumsi mata uang. Deskripsi dan thumbnail tersedia lengkap pada seed.http. Menjalankan seed berulang akan membuat record tambahan; periksa hasil sebelum mengulangi.

Jika pengisian dilakukan manual lewat New record, gunakan kelima data pada seed.http satu kali. Setelah modifikasi skema, lakukan Settings > Export collections dan simpan hasil sesuai B16.

## B11 - account-service: users dan favorites (slide 45)

Dashboard account-service :8092 harus memiliki `users` bertipe Auth dan `favorites` bertipe Base.

`users` memiliki email, password, name, dan avatar. Periksa pendaftaran publik: Create Rule terbuka dan kosong. Password authentication aktif dengan identity email. Skema users dari ZIP mengatur akses data user berdasarkan ID user yang login.

| Field favorites | Tipe | Pengaturan |
|---|---|---|
| user | Relation ke users | Single / maxSelect 1, Nonempty, cascade delete |
| product_id | Plain text | Nonempty, berisi ID produk dari product-service |

Pada tab Indexes, periksa unique index pasangan user dan product_id:

```sql
CREATE UNIQUE INDEX idx_fav_user_product ON favorites (user, product_id)
```

| Rule favorites | Nilai persis |
|---|---|
| List | user = @request.auth.id |
| View | user = @request.auth.id |
| Create | @request.auth.id != "" && @request.body.user = @request.auth.id |
| Update | terkunci / null |
| Delete | user = @request.auth.id |

User hanya bisa membaca favorit miliknya, menambah favorit untuk ID dirinya, dan menghapus favorit miliknya. Produk yang sama tidak bisa difavoritkan dua kali oleh user yang sama. `product_id` berupa teks lintas service dan bukan relation SQLite ke products.

## B12 - Kontrak API lewat Gateway (slide 46)

| Method | Endpoint setelah /api | Auth | Body / query | Hasil yang diharapkan |
|---|---|---|---|---|
| GET | /products | - | ?perPage=5&sort=-created | 200, items/page/totalItems |
| GET | /products | - | ?filter=title~'kopi' | 200, hasil pencarian |
| GET | /products/:id | - | - | 200 produk, 404 jika tidak ditemukan |
| POST | /auth/records | - | name, email, password, passwordConfirm | 200 akun baru, 400 jika email terpakai |
| POST | /auth/auth-with-password | - | identity, password | 200, token dan record |
| GET | /favorites | Bearer | - | 200, items milik user |
| POST | /favorites | Bearer | user, product_id | 200, 400 jika duplikat |
| DELETE | /favorites/:id | Bearer | - | 204, tanpa isi |

Bearer dikirim sebagai `Authorization: Bearer <token>`. ID pada DELETE adalah **ID record favorites**, bukan product_id. Token berasal dari login `users` di account-service, bukan token superuser.

## B13 - Tools untuk Menguji API (slide 47)

| Tool | Pekerjaan |
|---|---|
| Dashboard PocketBase | koleksi, data, rules, ekspor, backup |
| VS Code + REST Client (Huachao Mao) | seed.local.http dan api.local.http |
| Postman | import koleksi, pilih environment, Run collection |
| Browser | GET /api/products sebagai pemeriksaan cepat |
| aaPanel > Docker | status, log, dan pemantauan container |

Jika belum tersedia, pasang ekstensi REST Client melalui Extensions VS Code dan siapkan Postman Desktop atau ekstensi Postman sesuai modul. Laptop harus dapat mengakses IP STB pada jaringan yang sama.

## B14 - REST Client: Uji API dari VS Code (slide 48)

Di **PowerShell laptop**, pada folder proyek lokal:

```powershell
Copy-Item -LiteralPath .\api.http -Destination .\api.local.http
```

Buka api.local.http. Ganti IP_SERVER pada @baseUrl dengan alamat server Anda. Isi `@testName`, `@testEmail`, dan `@testPassword`. Nama Budi dan alamat budi@itb.ac.id pada template berasal dari contoh modul; pilih akun uji Anda sendiri. Gunakan password baru minimum 8 karakter, sesuai skema users. Password admin dan password akun uji adalah dua nilai berbeda perannya.

Jalankan Send Request berurutan:

| No. | Request | Pemeriksaan |
|---|---|---|
| 1 | GET /health | HTTP 200, gateway=ok |
| 2 | GET /products?perPage=5&sort=-created | HTTP 200, ada lima produk; productId terisi |
| 3 | GET /products?filter=title~'kopi' | HTTP 200, Kopi Gayo muncul |
| 4 | GET /products/{{productId}} | HTTP 200, ID sama dengan produk terpilih |
| 5 | POST /auth/records | HTTP 200 pada akun baru |
| 6 | POST /auth/auth-with-password | HTTP 200, token dan userId terisi |
| 7 | POST /favorites | HTTP 200, user dan product_id sesuai |
| 8 | GET /favorites | HTTP 200, record favorit milik user tampil |
| 9 | DELETE /favorites/{{favId}} | HTTP 204, body kosong |

productId berasal dari respons request bernama list; token/userId berasal dari login; favId berasal dari request bernama favs. Jalankan kembali request 8 setelah penghapusan untuk memastikan favorit uji sudah hilang.

Jika hasil favorit berisi product_id berupa teks {{list.response.body.$.items[0].id}}, variabel request belum terselesaikan. Jalankan kembali request 2, 4, dan 6 pada file yang sama sebelum request 7-9. Cocokkan product_id pada favorit dengan id produk aktual; HTTP 200 saja belum membuktikan relasi produk benar.

Pada pengulangan, registrasi dapat menghasilkan 400 karena email sudah ada. Baca body respons untuk memastikan penyebabnya, lalu login ke akun tersebut. Tambah favorit duplikat juga dapat menghasilkan 400. Error validasi lain perlu diperbaiki sesuai field yang dilaporkan.

## B15 - Postman: Import Koleksi Siap Pakai (slide 49)

1. Import `postman/KatalogKu-PocketBase.postman_collection.json`.
2. Import environment Lokal, ServerLAN, dan Cloudflare dari folder postman/.
3. Pilih **KatalogKu - ServerLAN**. Ganti IP_SERVER pada baseUrl dengan alamat server Anda: `http://IP_SERVER:8080/api`.
4. Pada environment di aplikasi Postman, tambahkan nilai lokal `testName`, `testEmail`, dan `testPassword`. Gunakan akun uji yang sama jika akan melanjutkan hasil REST Client.
5. Jalankan sembilan request dari atas ke bawah. Script referensi menyimpan productId, token, userId, dan favId sebagai collection variables.
6. Klik kanan koleksi > Run, periksa sembilan request beserta hasil test.

**Cara membaca test referensi:** test registrasi dan tambah favorit pada ZIP menerima HTTP 200 atau 400 untuk mengakomodasi pengulangan. Test hijau pada dua request tersebut belum membuktikan operasi berhasil. Untuk akun/favorit baru, pastikan HTTP 200 dan respons record benar. Jika 400, pastikan penyebabnya benar-benar duplikat, login berhasil, favorit memang ada, dan request berikutnya memakai ID yang tepat. Simpan ringkasan Run yang memperlihatkan seluruh urutan.

Environment Lokal tetap menggunakan localhost untuk penggunaan lokal. Environment Cloudflare masih merupakan contoh Bagian C dan belum merupakan URL API yang aktif. Saat mengekspor ulang koleksi setelah uji, kosongkan token, userId, productId, favId, dan password sebelum memasukkannya ke Git.

## B16 - Alur Kerja Harian dengan PocketBase (slide 50)

Alur mengikuti modul: ubah koleksi/rule di dashboard, uji lewat REST Client/Postman, ekspor skema, commit, lalu import skema pada deployment tujuan. Perubahan koleksi dan API Rules langsung berlaku tanpa rebuild image.

### 1. Ekspor skema aktual dari STB

1. Dashboard product-service > Settings > Export collections. Pilih products dan simpan sebagai `pb_schema/product_collections.json` pada proyek laptop.
2. Dashboard account-service > Settings > Export collections. Pilih users dan favorites lalu simpan sebagai `pb_schema/account_collections.json` pada proyek laptop.
3. Periksa field, rules, relation, dan unique index hasil ekspor.

Berkas di pb_schema saat ini merupakan **hasil Export dari instance STB yang dikerjakan**, sesuai kriteria pengumpulan. Pada perubahan berikutnya, ekspor ulang konfigurasi aktual dan simpan kembali pada dua nama file tersebut.

### 2. Commit dan push hasil ekspor

Di **PowerShell laptop**, pada checkout repository:

```powershell
git status --short
git diff -- pb_schema
git add pb_schema/product_collections.json pb_schema/account_collections.json
git diff --cached --name-only
git commit -m "Export skema PocketBase Bagian B dari STB"
git push
```

Periksa daftar file sebelum commit. `.env`, file *.local.http, token, password, dan backup database tidak menjadi bagian repository. `.gitignore` sudah mencakup jenis berkas lokal tersebut. Jika tidak ada perubahan skema, Git dapat melaporkan working tree bersih; simpan bukti ekspor dan cocokkan file yang ada.

### 3. Backup

Pada masing-masing dashboard, buka Settings > Backups, buat backup, lalu unduh. Simpan backup di lokasi pribadi di luar repository. Hasil backup berisi data dan harus diperlakukan berbeda dari ekspor definisi koleksi.

PocketBase mendukung hooks JavaScript apabila diperlukan logika khusus. Pada jalur modul ini, backend tetap memakai konfigurasi PocketBase. Versi Node.js + Express merupakan lampiran terpisah.

## Bukti penyelesaian Bagian B

Gunakan CHECKLIST_B.md selama pelaksanaan. Ketentuan pengumpulan pada slide 63 Full PPT meminta repository berisi Compose, gateway, serta pb_schema hasil Export, tanpa .env. Laporan akhir seluruh Week 7 bernama `NIM_W07_Microservice.pdf` dan menampung bukti bagian-bagian tugas yang sudah dikerjakan.

Untuk menulis laporan Bagian B berdasarkan pelaksanaan nyata, kumpulkan:

- status tiga service Bagian B dan pemeriksaan gateway;
- dashboard products dengan data dan rules;
- dashboard users/favorites dengan relation, unique index, dan rules;
- hasil sembilan request REST Client dan Run Postman;
- hasil Export collections dan commit repository.

Daftar bukti rinci ini membantu mendokumentasikan B1-B16. Ketentuan eksplisit pengumpulan Bagian B tetap repository yang dijelaskan slide 63. HTTPS Cloudflare, integrasi APK, dan pengembangan review-service memiliki butir tersendiri pada Full PPT dan tidak dijadikan klaim hasil Bagian B ini.

## Acuan

- Guide Bagian B: halaman 1-18, nomor slide 33-50.
- Full PPT Guide TST: slide 33-50 untuk materi B dan slide 63 untuk pengumpulan.
- Hasil Bagian A: bukti setup STB, SSH, aaPanel, Docker, MySQL, dan phpMyAdmin.
- ZIP katalogku-pocketbase dari dosen/asisten yang disediakan pengguna.
- [Dokumentasi Docker: daemon data directory dan containerd image store](https://docs.docker.com/engine/daemon/#daemon-data-directory), untuk pemeriksaan lokasi penyimpanan tambahan.
- [Dokumentasi PocketBase: API Rules](https://pocketbase.io/docs/api-rules-and-filters/), untuk memperjelas makna respons ketika rules tidak terpenuhi.

Dokumen dan ZIP referensi menjadi acuan tugas. Tautan dokumentasi resmi dipakai untuk pemeriksaan teknis tambahan, tanpa mengubah versi dan arsitektur modul.

Catatan import dashboard diverifikasi pada [kode sumber PocketBase v0.40.5](https://github.com/pocketbase/pocketbase/blob/v0.40.5/ui/src/settings/sync/pageImportCollections.js).

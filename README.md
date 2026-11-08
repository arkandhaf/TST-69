# KatalogKu API - Bagian B TST

Backend KatalogKu dengan PocketBase **0.40.5**, gateway Nginx **1.28**, dan Docker Compose. Konfigurasi runtime serta skema awal berasal dari ZIP referensi modul.

Baca **[PANDUAN_B_STB.md](PANDUAN_B_STB.md)** untuk langkah B1-B16 dan gunakan **[CHECKLIST_B.md](CHECKLIST_B.md)** untuk mencatat bukti pelaksanaan.

| Service | Tugas | Akses pada STB |
|---|---|---|
| gateway | routing API | http://IP_SERVER:8080/api |
| product-service | products, SQLite terpisah | http://IP_SERVER:8091/_/ |
| account-service | users dan favorites, SQLite terpisah | http://IP_SERVER:8092/_/ |

## Status pengerjaan

Konfigurasi dan skema telah diterapkan pada STB. Lima produk contoh tersedia, alur REST Client berhasil, serta Postman Collection Runner menghasilkan 9 Passed, 0 Failed, dan 0 Errors. Registrasi ulang menghasilkan 400 dengan data.email.code = validation_not_unique; login dan alur favorit berhasil. Kedua berkas pb_schema merupakan hasil Export collections dari instance yang dikerjakan.

Sebelum build, periksa arsitektur server, versi Compose, ketersediaan port, dan lokasi serta ruang penyimpanan Docker.

## Pengujian dan password

- `api.http` dan environment Postman ServerLAN menggunakan placeholder IP_SERVER; ganti pada salinan lokal dengan alamat server Anda.
- Salin `api.http` menjadi `api.local.http`, lalu isi testName, testEmail, dan testPassword pada salinan lokal.
- Salin `seed.http` menjadi `seed.local.http`, lalu isi adminPassword pada salinan lokal.
- Untuk Postman, isi testName/testEmail/testPassword pada environment lokal di aplikasi dan pilih ServerLAN.
- Password dikosongkan pada template. Nama/email Budi mengikuti contoh modul dan perlu disesuaikan dengan akun uji.
- `.env`, salinan *.local.http, password asli, token, serta backup database tidak masuk repository.

Koleksi Postman berisi sembilan request seperti referensi. Test registrasi dan tambah favorit menerima HTTP 200/400 untuk pengulangan. Baca body respons untuk membedakan duplikat dari kegagalan validasi lain.

## Isi proyek

```text
docker-compose.yml
pocketbase/Dockerfile
gateway/nginx.conf
pb_schema/product_collections.json
pb_schema/account_collections.json
api.http
seed.http
postman/
.env.example
.gitignore
PANDUAN_B_STB.md
CHECKLIST_B.md
```

Cloudflare Tunnel disertakan ZIP sebagai profile opsional untuk Bagian C. Perintah Compose biasa menjalankan tiga service Bagian B.

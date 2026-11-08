# Checklist pelaksanaan Bagian B

Status pelaksanaan: konfigurasi, koleksi, skema hasil ekspor, REST Client, dan Postman telah diperiksa. Backup pribadi dicatat setelah pelaksanaan dikonfirmasi.

| Urutan | Kriteria | Selesai | Bukti / catatan aktual |
|---|---|---|---|
| Pembuka | Menjelaskan monolith dan microservice | [x] | |
| B1 | PocketBase 0.40.5, SQLite, dashboard, REST API, auth/rules | [x] | |
| B2 | gateway, product-service, account-service; data terpisah | [x] | |
| B3 | Alur login-token-favorit dan komposisi detail produk | [x] | |
| B4 | Struktur folder lengkap | [x] | |
| B5 | Dockerfile alpine 3.22, PB 0.40.5, ARM64, /pb_data | [x] | |
| B6 | Compose, dua volume, port 8080/8091/8092 | [x] | |
| B7 | Routing health, products, auth, favorites; nginx -t | [x] | |
| B8 | Build selesai, tiga service Up, dua superuser dibuat | [x] | |
| B9 | Semua field products serta List/View publik, mutasi admin | [x] | |
| B10 | Merge with the existing collections aktif; preview tidak menghapus koleksi internal | [x] | |
| B10 | Import skema ke service yang sesuai, lima produk terisi | [x] | |
| B11 | users Auth; favorites relation, required, cascade, unique index | [x] | |
| B11 | Rules hanya memberikan akses favorit kepada pemiliknya | [x] | |
| B12 | Endpoint, body, auth, respons sesuai kontrak PocketBase | [x] | |
| B13 | Dashboard, REST Client, Postman, browser, aaPanel digunakan sesuai peran | [x] | |
| B14-1 | Gateway 200 | [x] | |
| B14-2 | List produk 200 dan productId valid | [x] | |
| B14-3 | Pencarian kopi 200 dan produk ditemukan | [x] | |
| B14-4 | Detail produk 200 dan ID sesuai | [x] | |
| B14-5 | Register 200 untuk akun baru / duplikat 400 diperiksa | [x] | |
| B14-6 | Login 200 dengan token dan userId | [x] | Jangan menyalin token ke catatan |
| B14-7 | Tambah favorit 200 / duplikat 400 diperiksa | [x] | |
| B14-8 | List favorit 200, user dan product_id benar | [x] | |
| B14-9 | Delete favorit 204 dan hilang saat list ulang | [x] | |
| B15 | Import koleksi/env, pilih ServerLAN, Run sembilan request | [x] | |
| B15 | Test 200/400 dibaca bersama body, bukan warna saja | [x] | |
| B16 | Export products serta users/favorites dari instance STB aktual | [x] | |
| B16 | Hasil ekspor disimpan di pb_schema dan di-commit ke GitHub | [x] | |
| B16 | Backup dibuat/diunduh dari masing-masing service | [ ] | Simpan di luar repository |
| Pengumpulan B | Compose, gateway, pb_schema hasil Export tersedia di GitHub | [x] | https://github.com/arkandhaf/TST-69 |
| Pengumpulan B | .env, password asli, token, dan backup tidak di-commit | [x] | |

## Catatan pengujian aktual

Isi setelah pelaksanaan: tanggal, status build, status container, kode HTTP, kendala, dan tautan commit hasil ekspor. Jangan mengisi hasil sukses berdasarkan contoh pada panduan. Screenshot dan keluaran terminal untuk laporan harus berasal dari pelaksanaan nyata; tutupi password/token yang mungkin tampil.

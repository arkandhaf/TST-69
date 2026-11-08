# Checklist pelaksanaan Bagian B

Status awal: konfigurasi dan template uji tersedia; tanda centang di bawah hanya diisi setelah pekerjaan atau bukti terkait benar-benar diperiksa pada STB.

| Urutan | Kriteria | Selesai | Bukti / catatan aktual |
|---|---|---|---|
| Pembuka | Menjelaskan monolith dan microservice | [ ] | |
| B1 | PocketBase 0.40.5, SQLite, dashboard, REST API, auth/rules | [ ] | |
| B2 | gateway, product-service, account-service; data terpisah | [ ] | |
| B3 | Alur login-token-favorit dan komposisi detail produk | [ ] | |
| B4 | Struktur folder lengkap | [ ] | |
| B5 | Dockerfile alpine 3.22, PB 0.40.5, ARM64, /pb_data | [ ] | |
| B6 | Compose, dua volume, port 8080/8091/8092 | [ ] | |
| B7 | Routing health, products, auth, favorites; nginx -t | [ ] | |
| B8 | Build selesai, tiga service Up, dua superuser dibuat | [ ] | |
| B9 | Semua field products serta List/View publik, mutasi admin | [ ] | |
| B10 | Import skema ke service yang sesuai, lima produk terisi | [ ] | |
| B11 | users Auth; favorites relation, required, cascade, unique index | [ ] | |
| B11 | Rules hanya memberikan akses favorit kepada pemiliknya | [ ] | |
| B12 | Endpoint, body, auth, respons sesuai kontrak PocketBase | [ ] | |
| B13 | Dashboard, REST Client, Postman, browser, aaPanel digunakan sesuai peran | [ ] | |
| B14-1 | Gateway 200 | [ ] | |
| B14-2 | List produk 200 dan productId valid | [ ] | |
| B14-3 | Pencarian kopi 200 dan produk ditemukan | [ ] | |
| B14-4 | Detail produk 200 dan ID sesuai | [ ] | |
| B14-5 | Register 200 untuk akun baru / duplikat 400 diperiksa | [ ] | |
| B14-6 | Login 200 dengan token dan userId | [ ] | Jangan menyalin token ke catatan |
| B14-7 | Tambah favorit 200 / duplikat 400 diperiksa | [ ] | |
| B14-8 | List favorit 200, user dan product_id benar | [ ] | |
| B14-9 | Delete favorit 204 dan hilang saat list ulang | [ ] | |
| B15 | Import koleksi/env, pilih ServerLAN, Run sembilan request | [ ] | |
| B15 | Test 200/400 dibaca bersama body, bukan warna saja | [ ] | |
| B16 | Export products serta users/favorites dari instance STB aktual | [ ] | |
| B16 | Hasil ekspor disimpan di pb_schema dan di-commit ke GitHub | [ ] | |
| B16 | Backup dibuat/diunduh dari masing-masing service | [ ] | Simpan di luar repository |
| Pengumpulan B | Compose, gateway, pb_schema hasil Export tersedia di GitHub | [ ] | https://github.com/arkandhaf/TST-69 |
| Pengumpulan B | .env, password asli, token, dan backup tidak di-commit | [ ] | |

## Catatan pengujian aktual

Isi setelah pelaksanaan: tanggal, status build, status container, kode HTTP, kendala, dan tautan commit hasil ekspor. Jangan mengisi hasil sukses berdasarkan contoh pada panduan. Screenshot dan keluaran terminal untuk laporan harus berasal dari pelaksanaan nyata; tutupi password/token yang mungkin tampil.

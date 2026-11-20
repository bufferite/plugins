# Plugin komunitas

Registry resmi adalah katalog **read-only** dari repo
[`bufferite/plugins`](https://github.com/bufferite/plugins). Aplikasi hanya
membaca indeks yang sudah ditandatangani, menampilkan pencarian/detail, lalu
menginstal paket yang Anda pilih.

Tidak ada marketplace lanjutan, akun, rating, ulasan, laporan, atau backend
terpisah. Kontribusi plugin cukup lewat pull request ke repo registry.

## Alur kontribusi

1. Fork `bufferite/plugins`, lalu buat branch.
2. Tambahkan paket `.berp`:

   ```text
   packages/<pluginId>/<versi>/<nama>-<versi>.berp
   ```

3. Tambahkan satu entri ke `registry-source.json` (path, `summary`,
   `publishedAt`, `status`, `tags`).
4. Buka pull request. Workflow **Validate registry** memeriksa path, paket,
   signature sementara, izin, dan batas ukuran tanpa memakai secret.
5. Maintainer review source Lua/WASM dan izin memakai checklist
   `REGISTRY.md` §13.3.
6. Setelah merge, maintainer menandatangani indeks dan men-deploy. Setelah
   aplikasi me-refresh katalog, plugin muncul dan bisa dicari berdasarkan
   nama, `pluginId`, `summary`, atau tag.

Dari sisi kontributor, pekerjaan selesai saat PR di-merge. Tidak ada upload
manual ke server atau akun yang perlu dibuat.

## Isi pull request

- Paket `.berp` berisi `plugin.manifest.json`, entrypoint, dan `LICENSE`.
- Entri `registry-source.json` untuk paket tersebut.
- URL source repository dan lisensi di deskripsi PR (opsional, membantu
  review).
- Untuk tier Lua: source dapat dibaca, tanpa obfuscation atau bytecode
  dinamis.

Template PR menyediakan checklist singkat saat Anda membuka pull request.

## Status rilis

- `active` — tampil dan dapat diinstal.
- `deprecated` — masih dapat diinstal dengan peringatan.
- `yanked` — tidak ditawarkan untuk instalasi baru.
- `revoked` — dinonaktifkan otomatis di perangkat yang sudah memasangnya.

## Verifikasi lokal (opsional)

```powershell
dart run tool/registry_cli.dart validate package --in path/plugin.berp
```

CI tetap menjalankan verifikasi penuh. Perintah di atas hanya membantu
menangkap kesalahan lebih awal.

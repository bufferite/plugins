# Bufferite Plugin Registry (scaffold)

Folder ini adalah benih untuk repo registry publik
`github.com/bufferite/plugins`. Salin **isi** folder ini ke root repo
tersebut. Workflow di dalam folder ini tidak dijalankan dari repo aplikasi.

Repo aplikasi tetap `github.com/bufferite/bufferite`. Aplikasi tidak membaca
README atau `COMMUNITY.md`; satu-satunya sumber yang dibaca adalah
`registry.json` + `registry.json.sig` dari:

```text
https://plugins.bufferite.dev/registry/v1/registry.json
```

Status keputusan:

- **D12 selesai (2026-11-20):** GitHub Pages, kontribusi lewat PR, deploy
  atomik dari `main`.
- **D13 terbuka:** custody kunci Ed25519. Scaffold memakai secret environment
  sebagai bootstrap; sebelum registry dibuka publik, kunci indeks wajib
  dipindahkan ke offline/HSM.

## Struktur

```text
registry-source.json                 daftar paket yang dikurasi
packages/<pluginId>/<version>/*.berp  paket plugin
keys/registry_keys.json              trust store publik (tanpa private key)
COMMUNITY.md                         daftar repo komunitas, tidak dibaca app
.github/workflows/validate.yml       validasi PR tanpa secret
.github/workflows/publish.yml        tanda tangan + build + deploy Pages
```

Folder `packages/` dan `keys/` tidak menyertakan paket atau kunci produksi.
Isi keduanya saat mulai mempublikasikan plugin.

## Setup awal

1. Buat repo publik `bufferite/plugins` dan salin isi folder ini ke root-nya.
2. Aktifkan **Settings > Pages > Build and deployment > GitHub Actions**.
   Tambahkan custom domain `plugins.bufferite.dev`, lalu arahkan CNAME
   `plugins.bufferite.dev` ke `bufferite.github.io`.
3. Buat dua kunci Ed25519 dari root repo aplikasi. Jangan commit hasilnya:

   ```powershell
   dart run tool/registry_cli.dart keygen `
     --key-id official-package-2026-01 --usage package `
     --out .keys/package.json --public-out .keys/package-public.json

   dart run tool/registry_cli.dart keygen `
     --key-id official-index-2026-01 --usage index `
     --out .keys/index.json --public-out .keys/index-public.json
   ```

4. Gabungkan public key menjadi trust store publik dan commit ke repo
   registry:

   ```powershell
   jq -s '{keys: ((.[0].keys // []) + (.[1].keys // []))}' `
     .keys/package-public.json .keys/index-public.json `
     > ..\plugins\keys\registry_keys.json
   ```

5. Buat environment `registry-publish` di repo registry dengan required
   reviewers. Simpan isi file private key sebagai environment secret:

   - `REGISTRY_PACKAGE_KEY` — isi `package.json`
   - `REGISTRY_INDEX_KEY` — isi `index.json`

   Untuk bootstrap, keduanya boleh berupa secret environment. Kunci indeks
   produksi tetap harus pindah ke offline/HSM sesuai D13.

6. Set repository variable `BUFFERITE_TOOL_REF` ke commit SHA repo
   aplikasi yang sudah berisi `tool/registry_cli.dart` (F16-T08). Jangan
   memakai `main` untuk produksi.

7. Lindungi branch `main`: required review, status check
   `Validate registry`, no force-push, linear history. Batasi who can push.
   Aktifkan 2FA untuk owner org dan secret scanning.

## Alur kontribusi

1. Kontributor membuat branch/fork dan menambahkan paket:

   ```text
   packages/com.contoh.encoding-pack/1.2.0/encoding-pack-1.2.0.berp
   ```

2. Tambahkan entri ke `registry-source.json`:

   ```json
   {
     "path": "packages/com.contoh.encoding-pack/1.2.0/encoding-pack-1.2.0.berp",
     "summary": "Kumpulan operasi encoding tambahan.",
     "publishedAt": "2026-11-20T10:00:00Z",
     "status": "active",
     "tags": ["encoding"]
   }
   ```

3. Buka pull request. Workflow `validate.yml` berjalan **tanpa secret**:
   membuat kunci sementara, menandatangani paket, membangun indeks, lalu
   memverifikasi signature dan batasnya.
4. Maintainer review dengan checklist `REGISTRY.md` §13.3, termasuk source
   Lua/WASM, izin, lisensi, dan hash paket.
5. Setelah merge ke `main`, `publish.yml` menandatangani paket dengan kunci
   package, membangun indeks baru, menandatangani indeks, dan men-deploy
   seluruh site secara atomik.

File `.sig` tidak perlu di-commit. Workflow publish membuatnya saat deploy.
Untuk build lokal, tanda tangani paket lebih dulu dengan kunci package dev:

```powershell
dart run tool/registry_cli.dart sign package `
  --in packages/com.contoh.encoding-pack/1.2.0/encoding-pack-1.2.0.berp `
  --out packages/com.contoh.encoding-pack/1.2.0/encoding-pack-1.2.0.berp.sig `
  --key .keys/package.json
```

## Publikasi

`publish.yml` mengambil indeks publik terakhir sebagai `--base`, sehingga
`sequence` selalu naik. Workflow juga menolak perubahan pada versi yang sudah
pernah terbit: `sha256` dan `downloadUrl` harus sama. Ini menjaga URL paket
tetap immutable.

Urutan yang dilakukan workflow:

1. validasi source + paket,
2. tanda tangani setiap paket,
3. `build-index` dengan trust store publik,
4. tanda tangani `registry.json`,
5. verifikasi paket + indeks,
6. unggah artifact Pages dan deploy.

Jika deploy gagal, jangan menurunkan `sequence` dan jangan menimpa paket.
Perbaiki penyebabnya, lalu terbitkan indeks baru dengan `sequence` lebih
tinggi. Untuk mencabut versi, ubah `status` menjadi `yanked`/`revoked`.

## Keamanan

- Private key tidak pernah masuk repo, artifact, atau log.
- `keys/registry_keys.json` hanya berisi public key dan aman di-commit.
- Jangan memakai `pull_request_target` untuk menjalankan kode dari PR.
- Pin semua GitHub Actions ke commit SHA sebelum produksi. File scaffold ini
  memakai tag major agar mudah diperbarui saat awal.
- Gunakan environment reviewer untuk job yang memegang private key.
- Batasi akses tulis ke `main` dan `packages/**` lewat branch protection dan
  CODEOWNERS.
- GitHub Pages adalah single point of failure; aplikasi tetap wajib punya
  cache offline dan gagal dengan aman.

## Catatan D13

Workflow publish di scaffold menandatangani indeks dengan secret environment
`REGISTRY_INDEX_KEY`. Itu hanya jalan pintas bootstrap. Sebelum registry
dibuka publik, ganti langkah tersebut dengan prosedur signing offline/HSM dan
simpan signature bersama indeks untuk audit. Rotasi dan revocation mengikuti
`REGISTRY.md` §5.3.

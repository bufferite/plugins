# Bufferite Plugin Registry (scaffold)

Folder ini adalah benih untuk repo registry publik
`github.com/bufferite/plugins`. Salin **isi** folder ini ke root repo
tersebut. Workflow di dalam folder ini tidak dijalankan dari repo aplikasi.

Repo aplikasi tetap `github.com/bufferite/bufferite`. Aplikasi tidak membaca
README atau `COMMUNITY.md`; satu-satunya sumber yang dibaca adalah
`registry.json` + `registry.json.sig` dari:

```text
https://bufferite.github.io/plugins/registry/v1/registry.json
```

URL tersebut sementara (D15); domain kanonis jangka panjang adalah
`https://plugins.bufferite.dev/registry/v1/registry.json`.

Status keputusan:

- **D12 selesai (2026-11-20):** GitHub Pages, kontribusi lewat PR, deploy
  atomik dari `main`.
- **D15 selesai (2026-11-20):** memakai URL default GitHub Pages selama
  `bufferite.dev` belum terdaftar. Kembali ke domain kanonis lewat rilis
  aplikasi setelah DNS aktif.
- **D13 selesai (2026-11-20):** kunci indeks disimpan offline/air-gapped dan
  signing indeks dilakukan manual; kunci paket disimpan sebagai secret
  environment CI. Rotasi paket 90 hari, rotasi indeks 12 bulan, overlap kunci
  lama minimal 90 hari, revocation lewat rilis aplikasi yang memperbarui
  trust store.
- **D14 selesai (2026-11-20):** distribusi plugin hanya lewat PR ke repo
  GitHub ini. Marketplace lanjutan, akun, rating, ulasan, atau laporan
  ditunda; aplikasi hanya menampilkan katalog read-only dan pencarian lokal.

## Struktur

```text
registry-source.json                 daftar paket yang dikurasi
packages/<pluginId>/<version>/*.berp  paket plugin
keys/registry_keys.json              trust store publik (tanpa private key)
release/registry.json                indeks bertanda tangan (hasil signing offline)
release/registry.json.sig            signature indeks
.gitattributes                       jaga byte artifact bertanda tangan
COMMUNITY.md                         panduan kontributor, tidak dibaca app
.github/pull_request_template.md     checklist PR
.github/workflows/validate.yml       validasi PR tanpa secret
.github/workflows/publish.yml        tanda tangan paket + build indeks (artifact)
.github/workflows/deploy.yml         verifikasi + deploy indeks bertanda tangan
```

Folder `packages/` dan `release/` tidak menyertakan paket atau indeks produksi
di scaffold ini. `keys/` hanya berisi public key (aman di-commit); private key
tidak pernah ada di repo.

## Setup awal

1. Buat repo publik `bufferite/plugins` dan salin isi folder ini ke root-nya.
2. Aktifkan **Settings > Pages > Build and deployment > GitHub Actions**.
   Untuk sementara (D15), biarkan custom domain kosong agar site tersedia di
   `https://bufferite.github.io/plugins/`. Setelah `bufferite.dev` terdaftar,
   tambahkan custom domain `plugins.bufferite.dev` dan arahkan CNAME
   `plugins` ke `bufferite.github.io`, lalu rilis aplikasi dengan URL kanonis.
3. Buat kunci paket di mesin CI/administratif:

   ```powershell
   dart run tool/registry_cli.dart keygen `
     --key-id official-package-2026-11 --usage package `
     --out .keys/package.json --public-out .keys/package-public.json
   ```

4. Buat kunci indeks di mesin offline/air-gapped. Jangan pernah menyalin
   private key indeks ke mesin yang terhubung internet:

   ```powershell
   dart run tool/registry_cli.dart keygen `
     --key-id official-index-2026-11 --usage index `
     --out D:\offline\official-index-2026-11.json `
     --public-out D:\offline\official-index-public.json
   ```

5. Gabungkan public key menjadi trust store publik, lalu commit ke repo
   registry **dan** aplikasi:

   ```powershell
   jq -s '{keys: ((.[0].keys // []) + (.[1].keys // []))}' `
     .keys/package-public.json D:\offline\official-index-public.json `
     > ..\plugins\keys\registry_keys.json
   ```

   File yang sama dibundel aplikasi sebagai `assets/registry_keys.json`.

6. Buat environment `registry-publish` di repo registry dengan required
   reviewers. Simpan hanya kunci paket sebagai environment secret
   `REGISTRY_PACKAGE_KEY` (isi `package.json`). Kunci indeks **tidak** boleh
   menjadi secret CI.

7. Set repository variable `BUFFERITE_TOOL_REF` ke commit SHA repo aplikasi
   yang sudah berisi `tool/registry_cli.dart` (F16-T08). Jangan memakai `main`
   untuk produksi.

8. Lindungi branch `main`: required review, status check
   `Validate registry`, no force-push, linear history. Batasi who can push.
   Aktifkan 2FA untuk owner org dan secret scanning. Batasi juga akses ke
   folder `release/**` lewat CODEOWNERS.

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

3. Buka pull request; template PR berisi checklist singkat. Workflow
   `validate.yml` berjalan **tanpa secret**: membuat kunci sementara,
   menandatangani paket, membangun indeks, lalu memverifikasi signature dan
   batasnya.
4. Maintainer review dengan checklist `REGISTRY.md` §13.3, termasuk source
   Lua/WASM, izin, lisensi, dan hash paket.
5. Merge ke `main` = selesai untuk kontributor. `publish.yml` menandatangani
   paket dengan kunci package dan membangun indeks; maintainer lalu
   menandatangani indeks secara offline (lihat bagian berikut). Setelah deploy
   dan refresh katalog, plugin muncul di aplikasi.

Tidak ada marketplace lanjutan, akun, rating, atau laporan. Aplikasi hanya
melakukan pencarian lokal terhadap `registry.json` yang sudah ditandatangani.

File `.sig` paket tidak perlu di-commit. Workflow membuatnya saat build dan
deploy. Untuk build lokal, tanda tangani paket lebih dulu dengan kunci package
dev:

```powershell
dart run tool/registry_cli.dart sign package `
  --in packages/com.contoh.encoding-pack/1.2.0/encoding-pack-1.2.0.berp `
  --out packages/com.contoh.encoding-pack/1.2.0/encoding-pack-1.2.0.berp.sig `
  --key .keys/package.json
```

## Publikasi (signing offline)

`publish.yml` mengambil indeks publik terakhir sebagai `--base`, sehingga
`sequence` selalu naik. Workflow juga menolak perubahan pada versi yang sudah
pernah terbit: `sha256` dan `downloadUrl` harus sama. Ini menjaga URL paket
tetap immutable.

Urutan publish:

1. `publish.yml` memvalidasi source + paket, menandatangani paket, membangun
   `registry.json`, dan mengunggah artifact `registry-release` (14 hari).
2. Maintainer mengunduh artifact, memeriksa `registry.json.sha256`, lalu
   menandatangani `registry.json` dengan kunci indeks di mesin offline.
3. Maintainer memverifikasi signature dengan `keys/registry_keys.json`, lalu
   commit `release/registry.json` + `release/registry.json.sig`.
4. `deploy.yml` memverifikasi signature, mengambil indeks publik sebelumnya,
   membangun ulang indeks secara deterministik, dan membandingkannya
   byte-per-byte. Jika cocok, paket ditandatangani dengan kunci paket CI dan
   seluruh site di-deploy atomik.

Detail langkah signing ada di `release/README.md`. Jika deploy gagal, jangan
menurunkan `sequence` dan jangan menimpa paket. Perbaiki penyebabnya, lalu
terbitkan indeks baru dengan `sequence` lebih tinggi. Untuk mencabut versi,
ubah `status` menjadi `yanked`/`revoked`.

## Rotasi & revocation

- **Kunci paket:** rotasi setiap 90 hari. Tambahkan public key baru ke
  `keys/registry_keys.json` dan `assets/registry_keys.json` lewat rilis
  aplikasi, pasang private key baru sebagai `REGISTRY_PACKAGE_KEY`, lalu
  tandatangani ulang paket pada publish berikutnya. Pertahankan kunci lama
  berstatus `active` minimal 90 hari sebelum ditandai `revoked`.
- **Kunci indeks:** rotasi setiap 12 bulan. Buat kunci baru di mesin offline,
  pin public key lewat rilis aplikasi, dan pertahankan kunci lama minimal 90
  hari. Setelah semua indeks ditandatangani ulang, cabut kunci lama lewat
  rilis aplikasi berikutnya.
- **Revocation kunci:** ubah `status` menjadi `revoked` di trust store dan
  kirim lewat rilis aplikasi. Tidak ada remote key update; aplikasi lama tetap
  mempercayai kunci lama sampai diperbarui.
- **Revocation plugin:** ubah `status` entri menjadi `yanked`/`revoked` di
  `registry-source.json`, build ulang, dan tanda tangani indeks baru. Aplikasi
  menonaktifkan plugin `revoked` saat katalog dimuat/di-refresh.

## Keamanan

- Private key tidak pernah masuk repo, artifact, atau log.
- Kunci indeks hanya ada di mesin offline/HSM; CI tidak pernah menerimanya.
- `keys/registry_keys.json` dan `assets/registry_keys.json` hanya berisi public
  key dan aman di-commit.
- Jangan memakai `pull_request_target` untuk menjalankan kode dari PR.
- Pin semua GitHub Actions ke commit SHA sebelum produksi. File scaffold ini
  memakai tag major agar mudah diperbarui saat awal.
- Gunakan environment reviewer untuk job yang memegang private key.
- Batasi akses tulis ke `main`, `packages/**`, dan `release/**` lewat branch
  protection dan CODEOWNERS.
- GitHub Pages adalah single point of failure; aplikasi tetap wajib punya
  cache offline dan gagal dengan aman.

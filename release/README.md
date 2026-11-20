# Signed release index

Folder ini hanya berisi dua file yang dihasilkan dari signing offline:

```text
release/registry.json
release/registry.json.sig
```

Alur lengkap:

1. Merge perubahan paket ke `main`. Workflow **Build registry release**
   menandatangani paket, membangun `registry.json`, dan mengunggah artifact
   `registry-release`.
2. Unduh artifact, periksa `registry.json.sha256`, lalu salin
   `registry.json` ke mesin offline yang memegang kunci indeks.
3. Tanda tangani secara offline:

   ```bash
   dart run tool/registry_cli.dart sign index \
     --in registry.json \
     --out registry.json.sig \
     --key /media/offline/official-index-2026-11.json
   ```

4. Verifikasi signature dengan trust store publik sebelum membawa hasilnya
   kembali:

   ```bash
   dart run tool/registry_cli.dart verify index \
     --in registry.json --sig registry.json.sig \
     --keyring keys/registry_keys.json \
     --base-url https://bufferite.github.io/plugins/registry/v1/registry.json
   ```

5. Commit `registry.json` dan `registry.json.sig` ke folder ini. Workflow
   **Deploy signed registry** akan:
   - memverifikasi signature dengan trust store,
   - mengambil indeks publik sebelumnya,
   - membangun ulang indeks secara deterministik dari source + `generatedAt`
     dan membandingkannya byte-per-byte dengan file yang ditandatangani,
   - menandatangani paket dengan kunci paket CI,
   - men-deploy seluruh site secara atomik.

Jangan mengubah `registry.json` setelah ditandatangani. Jika ada perubahan
source atau paket, ulangi build dan signing. `generatedAt` dan `sequence`
harus berasal dari artifact build terbaru. File `.gitattributes` menonaktifkan
konversi line ending untuk artifact ini agar byte yang ditandatangani tidak
berubah saat checkout.

# Paket plugin

Simpan paket di path immutable:

```text
packages/<pluginId>/<version>/<nama-paket>.berp
```

Jangan menimpa paket yang sudah pernah terbit. Untuk perbaikan, terbitkan
versi baru.

File `.berp.sig` dibuat oleh CI saat publish dan tidak perlu di-commit.
Untuk build lokal, jalankan `sign package` lebih dulu (lihat `README.md`).

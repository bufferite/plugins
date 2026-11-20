# Trust store publik

File `registry_keys.json` berisi public key yang dipakai aplikasi dan CI untuk
memverifikasi signature registry. File ini **publik** dan aman di-commit.
Salin file yang sama ke `assets/registry_keys.json` di repo aplikasi.

Jangan pernah menaruh private key di folder ini.

Kunci aktif saat ini:

| keyId | usage | custody |
|---|---|---|
| `official-index-2026-11` | `index` | offline/air-gapped, signing manual |
| `official-package-2026-11` | `package` | secret environment `registry-publish` |

Format:

```json
{
  "keys": [
    {
      "keyId": "official-index-2026-11",
      "algorithm": "ed25519",
      "usage": "index",
      "publicKey": "base64-public-key",
      "status": "active",
      "createdAt": "2026-11-20"
    },
    {
      "keyId": "official-package-2026-11",
      "algorithm": "ed25519",
      "usage": "package",
      "publicKey": "base64-public-key",
      "status": "active",
      "createdAt": "2026-11-20"
    }
  ]
}
```

Buat file ini dari output `keygen --public-out` seperti dijelaskan di
`README.md`. Jangan mengisi `publicKey` dengan nilai contoh.

Rotasi dan revocation mengikuti `REGISTRY.md` §5.3 dan `README.md` bagian
"Rotasi & revocation". Menambah/mencabut kunci memerlukan rilis aplikasi baru
karena aplikasi memuat trust store dari aset, bukan dari jaringan.

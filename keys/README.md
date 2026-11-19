# Trust store publik

File `registry_keys.json` berisi public key yang dipakai aplikasi dan CI untuk
memverifikasi signature registry. File ini **publik** dan aman di-commit.

Jangan pernah menaruh private key di folder ini.

Format:

```json
{
  "keys": [
    {
      "keyId": "official-package-2026-01",
      "algorithm": "ed25519",
      "usage": "package",
      "publicKey": "base64-public-key",
      "status": "active",
      "createdAt": "2026-11-20"
    },
    {
      "keyId": "official-index-2026-01",
      "algorithm": "ed25519",
      "usage": "index",
      "publicKey": "base64-public-key",
      "status": "active",
      "createdAt": "2026-11-20"
    }
  ]
}
```

Buat file ini dari output `keygen --public-out` seperti dijelaskan di
`README.md`. Jangan mengisi `publicKey` dengan nilai contoh.

# Newline Normalizer

Plugin contoh resmi pertama untuk registry Bufferite. Operasi ini mengubah
akhir baris `CRLF` (Windows) dan `CR` (klasik) menjadi `LF`.

- Plugin ID: `dev.bufferite.newline-normalizer`
- Tier: `dsl`
- Operasi: `newline.normalize` — Normalisasi Baris Baru
- Izin: `cpu`
- Lisensi: MIT

## Contoh

Input (CRLF):

```text
baris satu\r\nbaris dua\r\n
```

Output (LF):

```text
baris satu\nbaris dua\n
```

Transformasi hanya menyentuh karakter akhir baris; isi dan encoding teks
tidak diubah.

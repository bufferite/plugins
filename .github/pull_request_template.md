## Plugin

- Plugin ID:
- Versi:
- Tier (dsl/lua/wasm):
- Source repository (opsional):
- Lisensi:

## Checklist

- [ ] Paket ada di `packages/<pluginId>/<version>/`.
- [ ] Entri baru ditambahkan ke `registry-source.json`.
- [ ] Workflow **Validate registry** lulus.
- [ ] Source Lua/WASM dapat direview; tidak ada obfuscation.
- [ ] Izin minimal dan dijelaskan di deskripsi PR.
- [ ] Tidak ada secret, token, atau data pribadi di paket.

## Catatan review

Jelaskan fungsi plugin dan izin yang diminta. Maintainer memakai checklist
`REGISTRY.md` §13.3 sebelum merge.

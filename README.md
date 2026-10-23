# Carousel Skills Bake-off

Testbed untuk membandingkan skill carousel publik (skills.sh / GitHub) dengan pipeline internal PPAI. Satu brief konten yang identik dijalankan di tiap skill, hasil dibandingkan apa adanya.

## Skill yang diuji (7)

| # | Skill | Sumber | Tipe | Installs |
|---|-------|--------|------|----------|
| 1 | blacktwist-carousel-writer | blacktwist/social-media-skills | Copy-only (teks slide) | 2.2K |
| 2 | marcolang-instagram-carousel | marcolang/marketing-skills | HTML preview → Playwright PNG 1080×1350 | 810 |
| 3 | charlie-gemini-carousel | charlie947/social-media-skills | Image-gen prompt (Gemini route) | 1.4K |
| 4 | guizang-social-card | op7418/guizang-social-card-skill | Social card Swiss/editorial (XHS/WeChat) | 5.2K |
| 5 | itchnski-threads-carousel | itchernetski/threads-carousel-claude-skill | Next.js preview, 12 tipe slide, multi-preset | 83 |
| 6 | ash-harris-carousel-generator | ash-harris/carousel-generator | HTML template → export.mjs/py PNG 1080×1440 | 72 |
| 7 | kostja94-visual-content | kostja94/marketing-skills | Spesifikasi visual guidance (tanpa generator) | 1.3K (pack) |

## Struktur

```
TEST_BRIEF.md            — brief identik untuk semua skill (materi: AI news 23 Okt 2026)
material/source-vault/   — sumber primer dari vault daily-ai-summary
skills/                  — 7 skill hasil clone + ekstraksi
runs/                    — output per skill (dibuat saat eksekusi test)
NOTES.md                 — catatan perbandingan (diisi setelah tiap run)
```

## Cara pakai

1. Baca `TEST_BRIEF.md`.
2. Jalankan satu skill terhadap brief yang sama, tulis output ke `runs/<skill-name>/`.
3. Catat temuan (kemudahan, kualitas copy, kualitas design, format deviasi) di `NOTES.md`.
4. Brand design system disiapkan terpisah oleh Hanif — skill design-dependent menunggu itu.

## Audit

Semua 7 skill discan dengan `audit_skill.py` (167 file): **0 findings** — tidak ada prompt injection, command injection, exfiltration, atau path traversal. `package.json` dicek manual: tanpa postinstall/preinstall script, deps wajar (playwright, next, react, html-to-image, jspdf).

## Lisensi

Tiap skill mempertahankan lisensinya masing-masing (lihat dir masing-masing). Brief dan catatan test: MIT.

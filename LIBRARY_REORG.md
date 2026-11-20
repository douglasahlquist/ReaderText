# Library reorganization — one folder per version

Working file for moving ReaderText from combined passage pages to one folder per version.
Update the Log at the bottom as steps finish.

## Layout

```
versions/
  collections.json                 collections, their levels, books (USFM codes for the Bible), version list
  gap-report.md                    written by gap_analysis.py
  bible/<key>/version.json         name, language, script, license, numbering, columns, coverage
  bible/<key>/<USFM>/<chapter>.tsv verse ⇥ text  (zh-cuv-s / zh-cuv-t: verse ⇥ spaced Hanzi ⇥ pinyin)
                                   in the edition's own numbering (hbo-wlc: Masoretic)
  quran/<key>/version.json
  quran/<key>/<NNN>.tsv            ayah ⇥ text  (the riwayat add ⇥ hafsStart ⇥ hafsEnd, e.g. 2:1)
schemes/kjv.json                   reference versification for the Bible (31,102 verses)
schemes/masoretic.json             Hebrew chapter counts + fromKJV map (1,961 verses renumbered)
schemes/kufan.json                 reference count for the Qur'an (Hafs/Kufan, 6,236)
bundles/ChineseReader.json         required / default / offered / hidden version keys
```

Version keys are `<language>-<edition>`: en-kjv, zh-cuv-s, zh-cuv-t, hbo-wlc, grc-sblgnt, ko-krv,
th-kjv, vi-1934, fr-martin, de-luther1912, nsk-naskapi, en-web, en-bbe; Qur'an: ar-hafs, ar-latn-hafs,
ar-shubah, ar-warsh, ar-qalun, ar-duri, ar-susi, ar-bazzi, ar-qunbul, en-pickthall (en-sahih: hidden, private).
`legacyID` in each version.json gives the old id (kjv, simplified, original, quran-arabic …).

## version.json coverage

```json
"coverage": { "books": { "GEN": "all", "PSA": "all", "MAL": "1-3" },
              "omitted": ["JHN.7.53", "JHN.8.1"],
              "additional": ["3JN.1.15"] }
```

- **books**: what the edition contains. A book not listed is outside its canon, not a gap.
- **omitted** (curated, kept on rerun): verses the edition leaves out on purpose. The app shows
  "omitted in this edition".
- A verse inside coverage, missing, and not in omitted: the app shows "missing — possibly a data error".
- **additional**: verses beyond the reference scheme (e.g. the CUV's 3 John 1:15).
- Notes can attach to any of these by pointer: `bible · <key> · BOOK.chapter.verse`.

## Bundles

`required` (always on, can't be removed from the default view), `default`, `offered`,
`hidden` (never listed, added or searched — licensing/marketing).

## Tools (ChineseReaderTools/library/)

1. `WORKSPACE_ISLAM=… python3 build_versions.py <ReaderText>` — builds the version folders from
   BibleSources (Chinese through the same language kit as the pages), the riwayat sources, and
   search/*.tsv for Naskapi, Hafs, romanization and Pickthall. Keeps curated coverage, notes, access.
2. `python3 gap_analysis.py <ReaderText>` — fills coverage (omitted with reasons, additional with
   reasons), writes schemes/ and versions/gap-report.md + .json. Seed reasons live in SEED.
3. `python3 gap_report_pdf.py <ReaderText> --islam <workspace_islam>` — documents/reference/
   bible-gap-report.pdf and islamic-texts-gap-report.pdf.
4. `python3 version_packs.py <ReaderText>` — one `<key>.pack` per version (the download, and later
   the encryption unit; sha in version.json "pack"), Chinese book names and Qur'an slugs in
   collections.json, versions/bible/paragraphs.json (the pages' groups). Run after gap_analysis.py.
5. `python3 patch_pages.py <ReaderText>` — brings the old combined pages in line with versions/
   (WLC through the Masoretic map, missing rows, gap notices, Malachi 4). Then rerun
   `search/search_index.py --library <ReaderText>` (it skips the notices).
`versification.py` holds the Masoretic↔KJV shifts.

## Steps

1. [x] Version folders, collections.json, bundles, schemes, gap tool (additive; app unaffected —
       `versions/` is not in AppConfig content.include).
2. [x] Fix data errors: Malachi 4, John 7:53, WLC alignment, riwayat counts (built from sources).
       Old pages patched to match (220 pages + Malachi 4); search index rebuilt.
3. [x] App reads the version folders (VersionLibrary.swift): installs each version from its pack
       (en-kjv, zh-cuv-s, zh-cuv-t ship in the app's Library.pack), makes whole-chapter pages from
       the installed versions (old page files when one in use isn't installed), and makes search
       files on the device. Bundle hidden list applied.
3b. [ ] Once every device runs that build: delete search/ from the library (and search_index.py),
       then the combined pages (bible/passages/*.wlc.md, *.kjv.md; editions pages later).
4. [~] App: gap notices shown muted in that version's row, skipped by Read Aloud and search.
       Next: notes on gaps (pointer bible · <key> · BOOK.c.v).
5. [x] Search built on the device from versions/ (VersionLibrary.searchText).
6. [ ] Restricted versions (gitignored now: versions/bible/nsk-naskapi, th-kjv, quran/en-sahih,
       search/naskapi.tsv, thkjv.tsv; their columns are still in the combined pages): AES-GCM encrypted per version, key via REST (API Gateway + Lambda + DynamoDB),
       cached in the Keychain. Applies to nsk-naskapi (permission), th-kjv (NC-ND), en-sahih (private).
7. [ ] Hadith editions and Other Documents as collections.
8. [ ] Remove old combined pages once all devices are on the new layout.

## Open data items (see documents/reference/bible-gap-report.pdf, islamic-texts-gap-report.pdf)

- fr-martin: 31 verses lost at French chapter breaks — re-source a complete Martin.
- ko-krv: 7 OT/NT verses to check; NT "(eopseum)" verses recorded as omitted — confirm with a print copy.
- nsk-naskapi: GEN 25:8, 25:15, PSA 134:1, JHN 7:53, COL 1:2 — check the workspace_naskapi corpus.
- Hadith: import as versions/hadith/<key>/ (book · section · number), gap-check per collection.
- en-asv, en-sahih: no data.

## Findings from the first gap report (2026-11-20, superseded)

- **Whole library**: Malachi 4 (6 verses) was never generated — the passage list follows the Hebrew
  3-chapter Malachi. John 7:53 is missing in every edition. Fix in the page builder.
- **hbo-wlc**: 134 gaps — the ~37 chapters where Hebrew numbering differs (Gen 31/32, Exod 8, Lev 6,
  Num 16, 1 Kgs 4, Joel 2–3 …). Needs a Masoretic→KJV mapping (or numbering "masoretic" plus a map).
- **grc-sblgnt**: critical-text omissions seeded as `omitted`. Still flagged: Rom 16:25–27 (present in
  SBLGNT — data error), Acts 19:41 and 2 Cor 13:14 (renumbered: 19:40, 13:13).
- **fr-martin** (38), **ko-krv** (26): mostly verses joined or renumbered; review and mark omitted or fix.
- **nsk-naskapi**: GEN 25:8, 25:15, PSA 134:1, COL 1:2 missing; Old Testament covers GEN, EXO, PSA only.
- **Qur'an riwayat** (Warsh, Qālūn, al-Dūrī, al-Sūsī, al-Bazzī, Qunbul): 54–64 ayat short of their
  counts — likely verse markers ﴿n﴾ merged into one line; re-split from the source.
- en-asv and en-sahih have no data in search/ yet.

## Log

- 2026-11-20: Step 1 done. 24 versions written (12,532 chapter files, ~71 MB uncompressed).
- 2026-11-20: Rebuilt from sources (all counts exact), Masoretic map, gap reasons, two gap-report PDFs,
  pages patched + Malachi 4, search index rebuilt, app reads the bundle and shows notices.
- 2026-11-20: Restricted folders kept out of git (first commit amended). Version packs; the app
  reads versions/ with page files as fallback, and searches on the device.

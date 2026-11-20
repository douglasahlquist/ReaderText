# Library reorganization — one folder per version

Working file for moving ReaderText from combined passage pages to one folder per version.
Update the Log at the bottom as steps finish.

## Layout

```
versions/
  collections.json                 collections, their levels, books (USFM codes for the Bible), version list
  gap-report.md                    written by gap_analysis.py
  bible/<key>/version.json         name, language, script, license, numbering, columns, coverage
  bible/<key>/<USFM>/<chapter>.tsv verse ⇥ text  (zh-cuv-s / zh-cuv-t: verse ⇥ text ⇥ pinyin)
  quran/<key>/version.json
  quran/<key>/<NNN>.tsv            ayah ⇥ text
schemes/kjv.json                   reference versification for the Bible (31,102 verses)
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

1. `python3 build_versions.py <ReaderText>` — splits the per-version search files into version folders.
   Keeps curated coverage.omitted, notes and access on rerun.
2. `python3 gap_analysis.py <ReaderText>` — fills coverage, writes schemes/ and versions/gap-report.md.

## Steps

1. [x] Version folders, collections.json, bundles, schemes, gap tool (additive; app unaffected —
       `versions/` is not in AppConfig content.include).
2. [ ] Fix data errors the gap report found (below), then rerun both tools.
3. [ ] App: read `versions/` (VersionStore from version.json + bundle; QuickRead/LessonView build
       comparative rows from chapter TSVs), keep old passage pages as fallback.
4. [ ] App: missing-verse notices from coverage; notes on gaps.
5. [ ] Search index built from versions/ instead of search/*.tsv.
6. [ ] Restricted versions: AES-GCM encrypted per version, key via REST (API Gateway + Lambda + DynamoDB),
       cached in the Keychain. Applies to nsk-naskapi (permission), th-kjv (NC-ND), en-sahih (private).
7. [ ] Hadith editions and Other Documents as collections.
8. [ ] Remove old combined pages once all devices are on the new layout.

## Findings from the first gap report (2026-11-20)

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

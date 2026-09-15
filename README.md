# all_comparisons — aligned 10-way test-set gallery

Qualitative comparison gallery for **AfterImage3D / cross-condition sparse-structure distillation**.

Live: **https://qiu-yuchen.github.io/all_comparisons/**

## What it shows

Rows are methods, each row is one 8-view render sheet. Every row uses the **same sample, the same
inference replicate (r0)** and the **same 8 view indices**, so cells align exactly across methods.

| # | Column | Baseline id |
|---|---|---|
| 1 | OURS (AfterImage3D) — fixed reference | `OURS-seed2027-step12000` |
| 2 | BASE | `BASE` |
| 3 | Teacher-II | `TEACHER-II` |
| 4 | Direct25 | `DIRECT25_APPLE` |
| 5 | LGM | `LGM_BIG_FIXROT_MVDREAM` |
| 6 | Shap-E | `SHAPE_TEXT300M` |
| 7 | LN3Diff | `LN3DIFF_T23D` |
| 8 | InstantMesh | `FLUX1DEV_INSTANTMESH_LARGE` |
| 9 | 3DTopia-XL | `THREEDTOPIA_XL_T23D` |
| 10 | Hunyuan3D-2 (FR40K accelerated) | `HUNYUAN3D2_FR40K_ACCEL_V1` |

## Scores

Two extra data files carry the numeric side, read straight from the frozen evaluation artefacts
(nothing recomputed, nothing zero-filled):

- `scores.json` — per-prompt `CLIP12 average` and `CLIP12 best-view` (plus Uni3D) for all 1536
  frozen prompts × the 10 methods above, keyed by `sample_id`, with `sample_meta` for split /
  category / gallery membership.
- `scores_aggregate.json` — the full aggregate set: overall and paired statistics, per-category and
  per-split breakdowns, OURS seed detail, and coverage.

The gallery page shows the current prompt's CLIP in every method header plus a per-prompt leaderboard.
`scores.html` (link: **全部得分结果**) is the full score board:

| tab | content |
|---|---|
| 总览 | every method × CLIP avg / CLIP best / Uni3D, with median, quartiles, std, coverage |
| 配对对比 | Δ = OURS − method with the archived paired-bootstrap 95% CI and win/loss/tie counts |
| 分类 / 划分 | CLIP avg broken down by the 8 categories and the 6 test splits |
| OURS 两种口径 | fixed reference seed vs post-hoc best-of-3-seed oracle, per-seed detail, seed-pick counts |
| 逐 prompt 明细 | sortable/searchable table of all 1536 prompts × 10 methods |
| 口径与来源 | definitions, provenance paths, quarantine and applicability notes |

### OURS under two conventions

| convention | branch | CLIP12 avg ×100 | CLIP12 best ×100 | Uni3D |
|---|---|---|---|---|
| **A — fixed reference (paper convention)** | `OURS-seed2027-step12000` | 30.2076 | 32.0135 | 0.1940 |
| **B — best-of-3-seed oracle (post-hoc upper bound)** | per-prompt per-metric max over seeds 2026/2027/2028 | 30.8518 | 32.6816 | 0.2071 |

Convention A is the only reference used for any claim; convention B is a post-hoc, metric-wise
oracle kept only as an upper bound and must not be quoted as an OURS result.

## Scope and caveats

- **1522 samples** = the intersection of the frozen 1536-prompt test set on which *all ten* methods
  have a complete 12-view render at replicate 0. Samples outside the intersection are not shown and
  are **not** substituted.
- **8 views** per sheet: `v00 0° · v01 30° · v03 90° · v04 120°` (top row),
  `v06 180° · v07 210° · v09 270° · v10 300°` (bottom row). Label burned into each tile.
- **Invalid results are kept and labelled**, not resampled or zero-filled
  (ours 6 / base 4 / teacher 3 cases; cause `fragmented_gaussian_render`).
- **Images only** — no 3D models, no videos.
- Renders are downscaled to 256×256 per view and packed into one WebP sheet per method
  (source renders are 512×512). This compression is **presentational only**; every numeric
  result in the paper is computed on the original uncompressed renders.
- The Hunyuan column is the **FR40K-accelerated** Paint configuration, not the original
  standard Paint. OURS' legacy ULIP numbers remain quarantined and are not used for any
  cross-method ranking on this page.

## Layout

```
index.html                gallery UI (search / split / category filters, per-method CLIP + validity badges)
scores.html               score board (all metrics, OURS two conventions, per-prompt detail)
manifest.json             sample list, method registry, validity map
scores.json               per-prompt CLIP/Uni3D for all 1536 prompts x 10 methods
scores_aggregate.json     aggregate statistics behind scores.html
img/<method>/<case>_r0.webp     one 1024x544 sheet per method per sample (1522 x 10 = 15,220 files)
```

Generated from the frozen unified name map + record name map of the comparison campaign
(`comparison_figure_handoff_20260915`) and from the archived per-prompt metric files of the
evaluation campaign. Read-only w.r.t. all experiment outputs.

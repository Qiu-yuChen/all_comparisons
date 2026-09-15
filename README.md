# all_comparisons — aligned test-set gallery (10 baselines + 3 OURS seeds)

Qualitative comparison gallery for **AfterImage3D / cross-condition sparse-structure distillation**.

Live: **https://qiu-yuchen.github.io/all_comparisons/**

## The gallery (`index.html`)

One **card per frozen prompt**, streamed in chunks, with sticky filters. Each card carries the
**original selected FLUX input**, a compact metric table, and one 8-view render sheet per method.
Every sheet uses the **same sample, the same inference replicate (r0)** and the **same 8 view
indices**, so cells align exactly across methods.

Filter by split / category / length bucket / complexity, search by `S` id, Original id or prompt,
and re-order the whole stream by **any core metric on any branch** (direction-aware: *higher*,
*lower* and *closer-to-1* metrics each start best-first, and missing values always sink to the
bottom). A method-visibility control switches between all 12 columns, the OURS/BASE/Teacher subset,
only the three seeds, only the external baselines, or a custom pick.

| # | Column | Baseline id | source |
|---|---|---|---|
| 1 | **OURS (seed2026)** | `OURS-seed2026-step12000` | H100 |
| 2 | **OURS (seed2027)** — fixed reference | `OURS-seed2027-step12000` | H100 |
| 3 | **OURS (seed2028)** | `OURS-seed2028-step12000` | H100 |
| 4 | BASE | `BASE` | H100 |
| 5 | Teacher-II | `TEACHER-II` | H100 |
| 6 | Direct25 | `DIRECT25_APPLE` | cloud |
| 7 | LGM | `LGM_BIG_FIXROT_MVDREAM` | cloud |
| 8 | Shap-E | `SHAPE_TEXT300M` | cloud |
| 9 | LN3Diff | `LN3DIFF_T23D` | cloud |
| 10 | InstantMesh | `FLUX1DEV_INSTANTMESH_LARGE` | cloud |
| 11 | 3DTopia-XL | `THREEDTOPIA_XL_T23D` | cloud |
| 12 | Hunyuan3D-2 (FR40K accelerated) | `HUNYUAN3D2_FR40K_ACCEL_V1` | owned 4090 |

### Three OURS seeds, shown without changing the convention

The paper convention is a **single fixed reference**, `OURS-seed2027-step12000` — no best-of-three,
no three-seed mean, no seed selection. The gallery now **displays** seed2026 and seed2028 as
first-class columns so the spread can be inspected, but this is a *presentation* change only:
every aggregate is still reported per-branch, and the reference column is unchanged. The overall
table at the top of the page lists the three seeds side by side plus the **three-seed range**, which
describes **training-seed sensitivity** and is not a degree of freedom to pick from.

The one result worth flagging plainly: after the ULIP correction, **BASE scores higher than all
three seeds** on corrected ULIP (0.0344 vs 0.0300 / 0.0318 / 0.0312). That is shown as-is.

## Scores

Three data bundles carry the numeric side, read straight from the frozen evaluation artefacts
(nothing recomputed, nothing zero-filled). The gallery page loads `scores.json` and also shows
the current prompt's CLIP in every method header plus a per-prompt leaderboard.

### `scores.json` / `scores_aggregate.json` — the original board

- `scores.json` — per-prompt `CLIP12 average` and `CLIP12 best-view` (plus Uni3D) for all 1536
  frozen prompts × the 10 methods above, keyed by `sample_id`, with `sample_meta` for split /
  category / gallery membership.
- `scores_aggregate.json` — the full aggregate set: overall and paired statistics, per-category and
  per-split breakdowns, OURS seed detail, and coverage.

`scores.html` (link: **CLIP / Uni3D 得分榜**) is the score board:

| tab | content |
|---|---|
| 总览 | every method × CLIP avg / CLIP best / Uni3D, with median, quartiles, std, coverage |
| 配对对比 | Δ = OURS − method with the archived paired-bootstrap 95% CI and win/loss/tie counts |
| 分类 / 划分 | CLIP avg broken down by the 8 categories and the 6 test splits |
| OURS 两种口径 | fixed reference seed vs post-hoc best-of-3-seed oracle, per-seed detail, seed-pick counts |
| 逐 prompt 明细 | sortable/searchable table of all 1536 prompts × 10 methods |
| 口径与来源 | definitions, provenance paths, quarantine and applicability notes |

### `metrics_v2.json` / `per_prompt_v2.json` — the extended metric board

`metrics.html` (**全指标总表**) covers every metric the campaign can defend, over 14 branches
(the 7 native branches *including* the three OURS training seeds, `BASE`, `SCALE-2K-R10`,
`TEACHER-II`, `TEACHER-IT`, plus the 7 external baselines). **Every column header sorts.**

| tab | content |
|---|---|
| 核心指标排名 | pick any metric → all 14 branches ranked by it, with median / quartiles / std / coverage, paired Δ vs the reference, win-loss-tie counts, and a scope filter (all / no-teacher / native 7 / external 7) |
| 全指标矩阵 | method × metric matrix grouped by family (语义 / Teacher 几何 / Teacher 图像 / 形状健康度 / 隔离), clickable columns |
| 配对对比 | Δ = OURS − method per prompt for any metric; the archived 95% CI where one exists |
| 逐 prompt 明细 | all 1536 prompts × 14 branches for 18 metrics, searchable, sortable by clicking any column |
| 口径与来源 | metric definitions, provenance, quarantine and applicability rules |

Sorting in every table is direction-aware — *higher-is-better*, *lower-is-better* and
*closer-to-1* metrics each start **best-first**, and missing values always sink to the bottom
without being zero-filled. (The `closer_to_1` metrics `volume_ratio_to_teacher` and
`extent_ratio_mean_to_teacher` are included in the per-prompt payload precisely so that path is
exercised.)

The gallery can also re-order the whole 1522-card stream by any core metric on any branch.

#### New in this board

| metric | what it is |
|---|---|
| **ULIP (per-item corrected)** | the quarantined ULIP column, recomputed with **one point cloud per forward pass**. The frozen run used `batch_size = 8` for the point branch, which perturbs PointBERT features (max abs deviation 0.2094 on the point branch vs 3.2e-7 on text). Sampler, normalisation, text encoder and checkpoint are byte-identical to the frozen pipeline; only the point batch size changed. Verified against an independent audit of 6 fixed prompts: worst \|Δ\| = 6.4e-7. **Native 7 branches only** — the 7 external baselines' ULIP was never recomputed and stays quarantined. |
| **Occupancy IoU / F1 / Precision / Recall vs Teacher** | voxel occupancy agreement with the `TEACHER-IT` branch. |
| **Chamfer vs Teacher** (4 alignment tiers + mesh) | native frame, translation-only, translation+scale, rotation-aligned, plus a 10 000-point surface Chamfer. |
| **Silhouette IoU / normal cosine / depth error vs Teacher** | 12-view geometry agreement. |
| **PSNR / SSIM / LPIPS vs Teacher** | per-azimuth image agreement with the `TEACHER-IT` render of the **same sample, replicate and azimuth**. |

#### Two caveats that matter

- **There is no ground-truth image in this protocol.** PSNR / SSIM / LPIPS therefore measure
  *agreement with the teacher's own render*, not fidelity to a dataset image. Every method uses an
  identical render contract (512 px, distance 2.0, elevation 20°, fov 40, near 0.8, far 1.6, white
  background), so the views are comparable by construction; a self-check scores `TEACHER-IT` against
  itself and returns SSIM 1.0 / LPIPS 0.0.
- **The teacher-referenced geometry group exists only for the 7 native branches.** The external
  baselines never had teacher geometry computed; their cells read **不适用 (not applicable)**, which
  is deliberately different from a missing value and from 0. Likewise the corrected ULIP reads
  **未修正** for the external baselines. `LGM` is a Gaussian representation, so the point metric
  (`Uni3D`) is **N/A**, not 0.

### OURS under two conventions

| convention | branch | CLIP12 avg ×100 | CLIP12 best ×100 | Uni3D |
|---|---|---|---|---|
| **A — fixed reference (paper convention)** | `OURS-seed2027-step12000` | 30.2076 | 32.0135 | 0.1940 |
| **B — best-of-3-seed oracle (post-hoc upper bound)** | per-prompt per-metric max over seeds 2026/2027/2028 | 30.8518 | 32.6816 | 0.2071 |

Convention A is the only reference used for any claim; convention B is a post-hoc, metric-wise
oracle kept only as an upper bound and must not be quoted as an OURS result.

### Confidence intervals

The archived paired bootstrap (10 000 reps) covers **only** `clip12_average` / `clip12_best` /
`uni3d` for the **7 external baselines**; the archive lists `ulip` under `excluded_metrics`. The
teacher geometry and image metrics were never paired-bootstrapped, and the native branches have no
archived intervals at all. Those rows show `—` and their Δ are descriptive. **No interval is
invented anywhere on these pages.**

## Scope and caveats

- **1522 samples** = the intersection of the frozen 1536-prompt test set on which *all ten* original
  methods have a complete 12-view render at replicate 0. Samples outside the intersection are not
  shown and are **not** substituted. The two extra OURS seed columns were added *after* this
  intersection was fixed and have all 1536 samples complete, so they do **not** shrink it — the
  cohort stays 1522, and the seeds appear over exactly the same prompts.
- **8 views** per sheet: `v00 0° · v01 30° · v03 90° · v04 120°` (top row),
  `v06 180° · v07 210° · v09 270° · v10 300°` (bottom row). Label burned into each tile.
- **Invalid results are kept and labelled**, not resampled or zero-filled. Validity is read from the
  **corrected authoritative per-result record** (`technical_valid`), *not* from the earlier handoff
  map, which is pre-correction and disagrees. Replicate-level invalid counts over the 1536 × 2 grid
  of the native branches — `BASE` 8 · `OURS-seed2026` 4 · **`OURS-seed2027` 7** (fixed reference) ·
  `OURS-seed2028` 2 · `TEACHER-II` 5 · `TEACHER-IT` 1 · `SCALE-2K-R10` 4 — all with cause
  `fragmented_gaussian_render`. Because the paired unit needs **both** replicates, a prompt with one
  invalid replicate shows no value (`—`) and **stays in the denominator**.
- **Images only** — no 3D models, no videos.
- Renders are downscaled to 256×256 per view and packed into one WebP sheet per method
  (source renders are 512×512). This compression is **presentational only**; every numeric
  result in the paper is computed on the original uncompressed renders.
- The Hunyuan column is the **FR40K-accelerated** Paint configuration, not the original
  standard Paint (its CLIP is FR40K-accelerated Paint; its Uni3D comes from the pre-simplification
  original Shape, so it does not describe the final FR40K mesh geometry). The legacy batched ULIP
  column remains quarantined for **every** branch; the per-item-corrected ULIP on `metrics.html`
  exists only for the 7 native branches and is labelled as a post-hoc engineering correction.

## Layout

```
index.html                gallery UI (card stream, FLUX input, 3 OURS seed columns, sticky filters,
                          core-metric re-ordering, method-visibility presets, lightbox)
scores.html               score board (CLIP / Uni3D, OURS two conventions, per-prompt detail)
metrics.html              extended metric board (ULIP / teacher geometry / PSNR-SSIM-LPIPS), all columns sortable
manifest.json             sample list, method registry, validity map (validity map is legacy — see caveats)
scores.json               per-prompt CLIP/Uni3D for all 1536 prompts x 10 methods
scores_aggregate.json     aggregate statistics behind scores.html
metrics_v2.json           29-metric registry, aggregates, paired deltas, archived CIs, provenance
per_prompt_v2.json        per-prompt detail behind metrics.html (1536 x 14 x 18)
img/<method>/<case>_r0.webp     one 1024x544 sheet per method per sample
                                (12 method dirs x 1522 = 18,264 files)
img/flux_refs/<case>_r0.webp    the original selected FLUX input per sample (1522 files, 512x512)
```

Generated from the frozen unified name map + record name map of the comparison campaign
(`comparison_figure_handoff_20260915`), the archived per-prompt metric files of the evaluation
campaign, the corrected structural re-analysis, a read-only teacher-referenced image-metric pass,
and a read-only contact-sheet pass over the frozen per-view renders for the two extra OURS seeds.
Read-only w.r.t. all experiment outputs.

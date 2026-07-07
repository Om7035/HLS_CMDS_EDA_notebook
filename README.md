# HLS-CMDS EDA

Exploratory data analysis of the HLS-CMDS heart/lung sound dataset (535 files:
50 heart-sound recordings, 50 lung-sound recordings, and 145 Mix triplets each
contributing a heart component, lung component, and mixed recording).

**Primary thesis:** the dataset's metadata describes more independent
recording scenarios than its audio actually contains. Exact byte-level file
reuse is substantial, crosses metadata contexts (including gender labels), and
has a direct, measurable consequence for naive downstream train/test
splitting.

Every number quoted below is computed live in the notebook, not asserted —
rerun it end-to-end and the printed outputs should match.

## Scope: EDA vs. Dataset Integrity & Leakage Audit

This repository contains two distinct layers of work, kept explicitly
separate in the notebook (Section 5 marks the boundary):

- **Layer 1 — Core EDA** (Sections 1–4.5, 16): environment/reproducibility
  setup, file inventory, metadata/label checks, audio technical integrity
  (clipping), signal-level and time-frequency exploration (waveforms,
  spectrograms, Mel-spectrograms, MFCCs), and a feature-engineering proposal
  for downstream classification. This directly answers the assigned EDA
  brief.
- **Layer 2 — Dataset Integrity & Leakage Audit** (Sections 5–15): the
  metadata/duplicate checks in Layer 1 surfaced that a large share of the
  corpus (305/535 files, 57%) participates in exact byte-identical duplicate
  groups, and that this duplication crosses supposedly-fixed labels including
  Gender. Rather than note that finding and move on, this layer quantifies
  its downstream consequence for ML use (train/test contamination under
  naive splitting), validates a mitigation (hash-aware splitting), and
  statistically tests a separate structural claim about the data (whether
  the dataset's own H+L→M pairing is acoustically meaningful), with
  correction for multiple comparisons.

Basic duplicate detection was already within the assigned brief ("check for
duplicate recordings"); what extends beyond it is the *depth* of that
investigation — exact byte-level hashing, empirical contamination
simulation, and hypothesis testing of the dataset's internal pairing claim.
That depth was not planned in advance — it follows directly from what
Section 5 found.

*One-line summary: the core EDA surfaced significant data-integrity issues;
the analysis was extended into a dataset-integrity and leakage audit,
followed by targeted statistical validation of assumptions relevant to ML
readiness.*

## Headline findings

- **344 distinct byte-level audio identities among 535 files** (SHA256 and
  MD5 grouping agree). 230 files are exact-hash-unique; 305 files fall into
  114 duplicate groups.
- **Naive random train/test splitting leaks data.** Pooled naive contamination
  averages ~51%, and is worse for single-task splits (~65% heart-only, ~74%
  lung-only), while a Mix-M-only task is near-zero. A hash-group-aware split
  eliminates this contamination entirely (0%), as measured by the exact-hash
  duplicate criterion used here.
- **41% of duplicate groups (47/114) contain an identical recording labeled
  under both genders** — Gender should not be treated as an acoustically
  grounded label for reused recordings.
- **H/L → M pairing test (hard-negative permutation test, 3 difficulty
  levels × 2 feature spaces = 6 tests):** the true (H, L) pair beats a
  mismatched same-context negative in 54–66% of triplets, with modest effect
  sizes (0.07–0.26). Under both Bonferroni and Benjamini–Hochberg correction
  for the 6 tests, the **Level-2 band-energy**, **Level-3 band-energy**, and
  **Level-1 MFCC** results remain significant; Level-1 band-energy and the
  Level-2/3 MFCC results do not survive correction. This establishes
  feature-domain association, not sample-level decomposability — it does not
  prove or disprove "M = H + L".
- **A short full-scale plateau (run ≥ 2 samples at the exact int16 ceiling)**
  is observed in 1/145 Mix-M files, consistent with possible clipping;
  36/145 files touch `|amplitude| >= 0.999` in isolation. This is suggestive,
  not a definitive clipping diagnosis.
- Dataset is synthetic/manikin-based: zero load failures, zero NaN/Inf,
  uniform format — a clean technical baseline, but with no guarantee of
  generalization to real, organically recorded clinical audio.

See [`notebooks/HLS_CMDS_EDA_notebook.ipynb`](notebooks/HLS_CMDS_EDA_notebook.ipynb)
for the full analysis, including the leakage simulation, the metadata schema
audit (Location field mixes heart-site and lung-field vocabularies), the
signal-level/spectral exploration (waveforms, STFT and Mel-spectrograms,
MFCCs), a feature-engineering and modeling proposal, and the final
synthesis/verdict table.

## Signal-level and spectral exploration

One representative file per role (HS, LS, Mix-H, Mix-L, Mix-M) is visualized
as a raw waveform, STFT spectrogram, Mel-spectrogram, and MFCC heatmap — see
[`outputs/figures`](outputs/figures) or Section 4.5 of the notebook. This is
exploratory/visual on a single file per role; the quantitative, citation-backed
heart-vs-lung frequency-band check across the *full* corpus is done separately
in Section 12.1 (and the two do not necessarily agree at N=1 — the notebook
is explicit about that).

The notebook also proposes a concrete feature set (band-energy ratios, MFCCs,
spectral centroid, zero-crossing rate), a fixed-length windowing/segmentation
strategy, and a suggested modeling path (classical ML baseline vs. CNN on
spectrograms) — see Section 16.

## Repo structure

```
.
├── notebooks/
│   └── HLS_CMDS_EDA_notebook.ipynb   # full analysis, run top-to-bottom
├── outputs/
│   ├── figures/                      # generated plots (PNG)
│   └── tables/                       # derived CSVs (manifest, issue log,
│                                      #   experiment results) — no raw audio
├── data/
│   └── README.md                     # where to get the raw dataset + SHA256 check
└── requirements.txt
```

## Running it

```bash
pip install -r requirements.txt
```

Then place the raw dataset per [`data/README.md`](data/README.md) and run
`notebooks/HLS_CMDS_EDA_notebook.ipynb` top-to-bottom (Jupyter or
`jupyter nbconvert --execute`). The notebook creates `data/raw`, `cache/`,
and `outputs/` as needed and validates the extracted file inventory before
proceeding.

## Caveats and scope

- Raw audio is not redistributed here (see `data/README.md`) — HLS-CMDS is a
  third-party dataset with its own source/licensing.
- Effect sizes throughout the H/L/M pairing analysis are small; this is
  reported as a modest, partially significant finding, not a strong result.
- The dataset is synthetic/manikin-generated; conclusions here should not be
  assumed to transfer to real clinical stethoscope recordings without
  further validation.

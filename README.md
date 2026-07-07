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

## Headline findings

- **344 distinct byte-level audio identities among 535 files** (SHA256 and
  MD5 grouping agree). 230 files are exact-hash-unique; 305 files fall into
  114 duplicate groups.
- **Naive random train/test splitting leaks data.** Pooled naive contamination
  averages ~51%, and is worse for single-task splits (~65% heart-only, ~74%
  lung-only), while a Mix-M-only task is near-zero. A hash-group-aware split
  eliminates this contamination entirely (0%).
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
audit (Location field mixes heart-site and lung-field vocabularies), and the
final synthesis/verdict table.

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

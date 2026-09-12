# Automated MC/DC Condition-Level Fault Localization Benchmark

This repository contains the verified dataset, automated pipeline, and experimental results for a condition-level Fault Localization (FL) benchmark built on Standard Infrastructure for Software Testing Research (SIR) C programs. The pipeline uses **Modified Condition/Decision Coverage (MC/DC)** probe injection and **Tarantula suspicion scoring** to localize faults down to individual Boolean sub-expressions.

---

## 📊 Dataset Metrics

The benchmark covers **6 SIR subjects**, resulting in **95 verified quality-controlled mutants** and **47 total injected condition probes**.

| Subject | Base MC/DC Pairs | Total Probes Injected | Verified Mutants | Mean Max Suspicion |
| :--- | :---: | :---: | :---: | :---: |
| `printtokens` | 31,829 | 4 | 7 | 0.7265 |
| `printtokens2` | 1,503,651 | 17 | 10 | 0.7897 |
| `schedule` | 414,883 | 6 | 8 | 0.5487 |
| `schedule2` | 1,458,061 | 12 | 9 | 0.6845 |
| `tcas` | 251,897 | 6 | 41 | 0.7873 |
| `totinfo` | 59,915 | 2 | 20 | 0.5549 |
| **Total / Overall** | **3,668,236** | **47** | **95** | **0.6819** |

---

## 📁 Repository Structure

* **`final_verified_mcdc_dataset.csv`**: Filtered 3-tier verified benchmark dataset containing non-zero suspicion scores and failing test suite cases.
* **`master_all_subjects_results.csv`**: Complete raw output dataset capturing all mutant executions and probe coverage matrices prior to filtering.
* **`mcdc_fault_localization_pipeline.ipynb`**: End-to-end Python/C execution pipeline that instruments decisions, compiles mutants, executes test suites, and generates Tarantula matrices.

---

## 🛠️ Reproduction & Usage

### 1. Raw Dataset Dependency
To re-run dataset generation from scratch, download the raw SIR subject C source files (`sir_dataset_master.zip`) and place the extracted folder in `/content/` before running the notebook.

### 2. Execution Environment
1. Open `mcdc_fault_localization_pipeline.ipynb` in Google Colab or a Linux environment with `gcc` installed.
2. Execute all pipeline cells. The script handles non-standard character encoding during subprocess outputs using `errors='replace'`.
3. Processed CSV output files will automatically be saved to `/content/mcdc_multi_subject_workspace/`.

---

## 🔬 Methodology & Quality Control

1. **Probe Injection**: Decision trees in C source files are parsed to inject condition probes tracking individual predicate evaluation outcomes.
2. **Suspicion Computation**: Dynamic execution traces compute Tarantula suspicion scores $S(e)$ for each probe $e$.
3. **3-Tier Quality Filtering**: Mutants are filtered to exclude uninformative runs (e.g., zero test suite failures, equivalent mutants, or non-executable probes).

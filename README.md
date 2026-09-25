# Sub-Decision Granularity in Spectrum-Based Fault Localization (SBFL)

This repository contains the implementation and experimental evaluation for studying **MC/DC-based sub-decision granularity in Spectrum-Based Fault Localization (SBFL)** using C programs from the **Software Infrastructure Repository (SIR)**.

The project investigates whether tracking individual Boolean conditions within compound decisions can provide finer-grained execution spectra and improve fault localization compared with traditional decision-level coverage.

---

## Overview

Traditional Spectrum-Based Fault Localization generally treats an entire decision as a single coverage entity.

For example:

```c
if (A && B)
```

is normally tracked as one decision.

This project introduces **MC/DC-based sub-decision instrumentation**, allowing the individual atomic conditions to be tracked separately:

```c
if (_mcdc_track(1, A) && _mcdc_track(2, B))
```

The resulting sub-condition execution spectra are then evaluated using established SBFL metrics.

The study compares:

- Standard decision-level coverage
- MC/DC sub-condition coverage
- Tarantula
- Ochiai
- D*
- Fault-localization rankings across SIR programs and mutants

---

## Experimental Results

The benchmark evaluation includes:

- **100 candidate mutants**
- **95 verified mutants**
- **93,267 base MC/DC condition pairs**
- **6 SIR subjects**
- **47 injected MC/DC probes**

### Overall Results

| Metric | Basic Decision-Level SBFL | MC/DC Sub-Decision SBFL | Relative Change |
|---|---:|---:|---:|
| Tarantula mean suspicion | 0.5545 | 0.6819 | +22.9% |
| Ochiai mean suspicion | 0.1670 | 0.2210 | +32.3% |
| D* mean suspicion | 6.1513 | 9.7293 | +58.2% |
| Mean fault inspection rank | 3.74 | 1.28 | 65.8% reduction |

Across the evaluated benchmark, the MC/DC-based sub-decision approach achieved a **94.7% overall win rate** when comparing the evaluated sub-decision spectra against standard decision-level spectra.

---

## SIR Benchmark Evaluation

| Subject | Evaluated Mutants | Verified Mutants | Base MC/DC Pairs | Injected Probes | Basic Rank | MC/DC Rank |
|---|---:|---:|---:|---:|---:|---:|
| printtokens | 7 | 7 | 5,479 | 4 | 2.50 | 1.00 |
| printtokens2 | 10 | 10 | 40,572 | 17 | 9.00 | 2.60 |
| schedule | 9 | 8 | 13,101 | 6 | 2.88 | 2.25 |
| schedule2 | 10 | 9 | 27,606 | 12 | 6.50 | 1.00 |
| tcas | 41 | 41 | 4,840 | 6 | 3.50 | 1.06 |
| totinfo | 23 | 20 | 1,669 | 2 | 1.50 | 1.00 |
| **Total / Overall** | **100** | **95** | **93,267** | **47** | **3.74** | **1.28** |

---

## Methodology

The experimental pipeline consists of the following stages:

```text
SIR Benchmark
      ↓
Source & Mutant Discovery
      ↓
AST-Based Condition Detection
      ↓
MC/DC Probe Instrumentation
      ↓
C Compilation
      ↓
Test Suite Execution
      ↓
Runtime Condition Traces
      ↓
Pass/Fail Spectrum Construction
      ↓
SBFL Metric Computation
(Tarantula / Ochiai / D*)
      ↓
Decision vs Sub-Decision Comparison
      ↓
Verification & Filtering
      ↓
Final Dataset & Visualizations
```

### 1. Source Discovery and Benchmark Parsing

The implementation scans SIR subject directories and identifies:

- Base source programs
- Mutant versions
- Test plans
- CLI test cases
- Relevant benchmark metadata

Expected benchmark subjects:

```text
sir_all_subjects/
├── printtokens/
├── printtokens2/
├── schedule/
├── schedule2/
├── tcas/
└── totinfo/
```

### 2. AST-Based Condition Detection

Compound Boolean expressions are identified in the C source code.

For example:

```c
if (A && B)
```

contains two atomic Boolean conditions:

```text
A
B
```

These conditions become individual instrumentation points.

### 3. MC/DC Probe Instrumentation

Each atomic Boolean condition is wrapped with an instrumentation function.

Example:

```c
if (A && B)
```

becomes conceptually:

```c
if (_mcdc_track(1, A) && _mcdc_track(2, B))
```

During execution, the instrumentation records the runtime value of each condition.

Runtime information is emitted using traces such as:

```text
MCDC_EVAL:<condition_id>:<value>
```

### 4. Automated Compilation and Execution

Instrumented mutant programs are compiled using GCC and executed against their corresponding SIR test suites.

The execution process records:

- Test pass/fail status
- Individual condition evaluations
- Runtime MC/DC traces
- Coverage information

### 5. SBFL Computation

The collected spectra are used to calculate three fault-localization metrics:

- Tarantula
- Ochiai
- D*

Each metric is evaluated using both:

1. Standard decision-level coverage
2. MC/DC sub-condition coverage

### 6. Verification and Filtering

Mutants are checked for:

- Successful compilation
- Successful test execution
- Valid failure behavior
- Valid runtime traces
- Usable coverage information

Only verified experiments are included in the final dataset.

---

## SBFL Formulations

Let:

- \(N_{cf}\) = number of failing tests covering component \(c_i\)
- \(N_{cp}\) = number of passing tests covering component \(c_i\)
- \(e_f\) = total number of failing tests
- \(e_p\) = total number of passing tests

### Tarantula

\[
S_{\text{Tarantula}}(c_i)
=
\frac{
\frac{N_{cf}(c_i)}{e_f}
}{
\frac{N_{cf}(c_i)}{e_f}
+
\frac{N_{cp}(c_i)}{e_p}
}
\]

### Ochiai

\[
S_{\text{Ochiai}}(c_i)
=
\frac{
N_{cf}(c_i)
}{
\sqrt{
e_f
\left(
N_{cf}(c_i)+N_{cp}(c_i)
\right)
}
}
\]

### D*

Using exponent \(2\):

\[
S_{D^*}(c_i)
=
\frac{
N_{cf}(c_i)^2
}{
N_{cp}(c_i)
+
(e_f-N_{cf}(c_i))
}
\]

---

## Repository Contents

```text
mcdc-fault-localization-benchmark/
│
├── MCDC_Fault_Localization_Benchmark (1).ipynb
├── README.md
├── LICENSE
└── .gitignore
```

The main notebook contains the benchmark discovery, instrumentation, compilation, execution, spectrum construction, SBFL computation, verification, analysis, and visualization pipeline.

---

## Benchmark Data

The experiments use programs from the **Software Infrastructure Repository (SIR)**.

The benchmark archive used during experimentation is:

```text
sir_all_subjects.zip
```

The benchmark data is treated as an external dependency and is not required to be committed directly to this repository.

After obtaining the benchmark archive, extract it so that the subject directories are available under:

```text
sir_all_subjects/
```

with the expected subjects:

```text
printtokens
printtokens2
schedule
schedule2
tcas
totinfo
```

The exact SIR distribution and its associated licensing or redistribution conditions should be followed when obtaining and using the benchmark.

---

## Requirements

### Operating System

A Linux/POSIX environment is recommended.

The pipeline can also be executed using WSL on Windows.

### Software

- Python 3.8+
- GCC
- Jupyter Notebook or JupyterLab

### Python Packages

```text
numpy
pandas
matplotlib
seaborn
jupyter
```

Install them using:

```bash
pip install numpy pandas matplotlib seaborn jupyter
```

A virtual environment is recommended:

```bash
python3 -m venv venv
source venv/bin/activate

pip install numpy pandas matplotlib seaborn jupyter
```

---

## Running the Experiment

Clone or obtain this repository and enter the project directory:

```bash
cd mcdc-fault-localization-benchmark
```

Activate the Python environment if one was created:

```bash
source venv/bin/activate
```

Start Jupyter:

```bash
jupyter notebook
```

Open:

```text
MCDC_Fault_Localization_Benchmark (1).ipynb
```

Ensure that the SIR benchmark directory is available at the location expected by the notebook before running the experiment.

---

## Generated Outputs

The pipeline produces experimental artifacts including:

```text
final_verified_mcdc_dataset.csv
```

The generated dataset contains the verified experimental results used for subsequent analysis.

Depending on the notebook configuration, additional outputs may include:

- Runtime condition traces
- SBFL suspicion scores
- Fault-localization rankings
- Verification statistics
- Comparison tables
- Visualization plots

---

## Experimental Design

The evaluation compares two levels of program representation.

### Baseline: Decision-Level Coverage

A compound Boolean decision is treated as one coverage entity.

Example:

```c
if (A && B)
```

is represented as a single decision.

### Proposed: MC/DC Sub-Decision Coverage

The individual atomic conditions are tracked separately.

Example:

```text
Decision
├── Condition A
└── Condition B
```

This produces a finer-grained execution spectrum that can distinguish which part of a compound decision participates in failing executions.

---

## Why MC/DC?

Modified Condition/Decision Coverage (MC/DC) is designed to determine whether individual conditions within a decision independently affect the decision outcome.

For a decision such as:

```c
if (A && B && C)
```

decision-level coverage observes the overall outcome, while MC/DC-oriented instrumentation provides information about the individual conditions:

```text
A
B
C
```

The project uses this additional granularity as the basis for evaluating finer-grained fault-localization spectra.

---

## Research Questions

The experimental framework is designed to investigate questions such as:

1. Does MC/DC-based sub-decision instrumentation provide more fine-grained execution information than standard decision-level coverage?

2. How does sub-decision granularity affect SBFL suspicion scores?

3. How do Tarantula, Ochiai, and D* behave under decision-level versus sub-decision spectra?

4. Does tracking individual Boolean conditions affect the resulting fault-localization rankings?

5. How consistently does the finer-grained representation identify locations associated with injected faults across different SIR subjects?

---

## Reproducibility

For reproducible experimentation:

1. Use the same SIR benchmark subjects.
2. Use the same mutant set.
3. Use the same test suites.
4. Run the instrumentation and execution pipeline without modifying generated traces.
5. Use the same SBFL formulations described above.
6. Preserve the verification and filtering procedure.
7. Compare the resulting rankings using the same aggregation procedure.

The notebook contains the implementation of the experimental workflow.

---

## Research Scope

This repository focuses specifically on:

- Spectrum-Based Fault Localization
- MC/DC-based instrumentation
- Sub-decision granularity
- Dynamic execution spectra
- C programs
- Mutation-based evaluation
- SIR benchmark programs
- Tarantula, Ochiai, and D* fault-localization metrics

It is intended as an experimental research artifact rather than a general-purpose fault-localization framework.

---

## License

This project is released under the MIT License. See [`LICENSE`](LICENSE) for details.

The SIR benchmark is an external research resource and is subject to its own terms and conditions.

---

## Citation

This repository is provided for research and experimental evaluation.

Citation information will be provided after the review process.

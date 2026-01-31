# PLINK2 GLM Testing Framework

This directory contains the continuous integration (CI) testing framework for validating PLINK2 GLM (Generalized Linear Model) implementations against R reference outputs.

## Overview

The testing framework runs PLINK2 GLM analyses on various datasets and compares the results with pre-computed R reference values to ensure statistical accuracy. Tests are organized into 12 batches for parallel execution in CI environments.

**Total Tests:** 108 test configurations across 12 batches (9 tests per batch)

**CI Automation:** Runs weekly via [.github/workflows/auto_checkcommits_tests_linuxmac.yml](/.github/workflows/auto_checkcommits_tests_linuxmac.yml) on both Linux and macOS, cycling through one batch per week.

## Quick Start

### Run All Tests (Batch 1)
```bash
# From repository root
./2.0/build_dynamic/CI-SCRIPTS/GLM_TESTS.sh
```

### Run Specific Batch
```bash
# Set batch number (1-12) and run
export GLM_BATCH_NUM=5
./2.0/build_dynamic/CI-SCRIPTS/GLM_TESTS.sh
```

### Local Testing
```bash
# Run with custom configuration
./2.0/build_dynamic/CI-SCRIPTS/LOCAL_PLINK2_GLM_TESTING.sh
```

## Main Scripts

### Entry Points

- **GLM_TESTS.sh**
  Main CI entry point. Sets up Python environment, creates results directory, and executes LOCAL_PLINK2_GLM_TESTING.sh.

- **LOCAL_PLINK2_GLM_TESTING.sh**
  Core test execution engine. Loads configuration, runs PLINK2 GLM for each test case, and validates results against R reference outputs.

### Configuration

- **PLINK2_GLM_TEST_CONFIG.sh**
  Central configuration file that defines:
  - Data paths (`datapath`, `outdir`)
  - Batch selection (`BATCH_NUM`)
  - Correlation threshold for pass/fail (`correlation_threshold`)
  - Loads test parameters from batch files

### Analysis & Validation

- **COMPARE_GLM_PLINK2_R.py**
  Python script that compares PLINK2 output with R reference results. Calculates correlations for betas, standard errors, and p-values.

- **checkglm.py**
  Utility for checking GLM output files.

- **glm_test_batches.py**
  Script for generating batch configuration files from master test list.

### Utilities

- **CLEANUP.sh**
  Removes test data, results, and cleans build artifacts.

- **DYNAMICBUILD_LINUXMAC_ROOT.sh**
  Build script for Linux/Mac dynamic builds.

## Test Configuration

### Batch System

Tests are divided into 12 batches stored in `CI_reference_files/`:
- `BATCH_01_PARAMS.txt` through `BATCH_12_PARAMS.txt`
- `ALL_BATCHES_MASTER_LIST.txt` - Complete list of all 108 tests

Each batch file contains ~9 test configurations.

### Parameter Format

Test parameters are CSV format:
```
dataset_name,model,n,covariate,threads
```

**Fields:**
- `dataset_name` - Dataset file prefix (e.g., `1kgp3_50k_nomiss_Av_nonintdose`)
- `model` - GLM model type: `linear`, `logistic`, or `firth`
- `n` - Sample subset size (1000, 32000, or 32017)
- `covariate` - Covariate name (e.g., `COV_1`) or blank for none
- `threads` - Number of threads for PLINK2 (1, 4, or 8)

**Example:**
```
1kgp3_50k_yesmiss_Av_nonintdose,linear,1000,COV_1,1
1kgp3_50k_nomiss_Av_nonintdose,logistic,32000,,4
```

### Test Conditions

**Datasets:**
- `1kgp3_50k_nomiss_Av_nonintdose` - No missing genotypes
- `1kgp3_50k_yesmiss_Av_nonintdose` - With missing genotypes

**Models:**
- `linear` - Uses phenotype `y`
- `logistic` - Uses phenotype `ybool`, no Firth correction
- `firth` - Uses phenotype `ybool`, hybrid Firth regression

**Sample Sizes:**
- 1000 - Small subset
- 32000 - Large subset (64 × 500)
- 32017 - Large non-64-multiple subset (64 × 500 + 17)

**Threading:**
- Single thread (--threads 1)
- Multi-threaded (--threads 4 or 8)

## Directory Structure

```
CI-SCRIPTS/
├── README.md                          # This file
├── GLM_TESTS.sh                       # Main CI entry point
├── LOCAL_PLINK2_GLM_TESTING.sh        # Core test runner
├── PLINK2_GLM_TEST_CONFIG.sh          # Configuration
├── COMPARE_GLM_PLINK2_R.py            # PLINK2 vs R comparison
├── glm_test_batches.py                # Batch file generator
├── checkglm.py                        # GLM output checker
├── CLEANUP.sh                         # Cleanup utility
├── DYNAMICBUILD_LINUXMAC_ROOT.sh      # Build script
├── CI_reference_files/                # Batch configurations
│   ├── BATCH_01_PARAMS.txt
│   ├── BATCH_02_PARAMS.txt
│   ├── ...
│   ├── BATCH_12_PARAMS.txt
│   └── ALL_BATCHES_MASTER_LIST.txt
├── results/                           # Test outputs (gitignored)
└── archive/                           # Deprecated scripts
    ├── GLM_TESTS_old.sh
    ├── GLM_TESTS_old2.sh
    ├── local_plink2_glm_testruns_old.sh
    └── ...
```

## Test Data Management

### Test Data Generation

Test data was generated and archived using [PLINK2SYN](https://github.com/physioforecast/PLINK2SYN), a genomics research pipeline for creating synthetic genotype-phenotype datasets.

**PLINK2SYN Pipeline:**
- **Purpose:** Generates synthetic GWAS datasets with known ground truth for validating PLINK2 statistical methods
- **Source Data:** 1000 Genomes Project Phase 3 (~50,000 biallelic SNPs)
- **Synthetic Samples:** ~32,000 augmented samples created by adding controlled Gaussian noise to real genotypes
- **Phenotypes:** Both quantitative and binary phenotypes with configurable heritability (~500 causal variants)
- **Reference Outputs:** Ground truth R GLM results for validation
- **Scenarios:** Datasets with and without missing genotypes for comprehensive testing

This pipeline ensures PLINK2 GLM implementations can be validated against known statistical outcomes from R, providing confidence in the accuracy of GWAS results.

### CI Test Data Caching

Test data is managed via [.github/workflows/cache_test_data.yml](/.github/workflows/cache_test_data.yml):

**How It Works:**
1. **Storage:** Test data hosted on Google Drive (File ID: `1lW5s8kwMeOwHPrbRoREtO5pwgKKK_8FR`)
2. **Cache Key:** `test-data-v1` - shared across all workflow runs
3. **Manual Trigger:** Run via workflow_dispatch in GitHub Actions UI
4. **Process:**
   - Downloads test_data.zip from Google Drive using `gdown`
   - Extracts to `test_data/` directory
   - Caches with GitHub Actions cache
   - Subsequent workflow runs restore from cache (no re-download)

**When to Run:**
- First-time setup of CI test data cache
- After updating test datasets (increment cache key to `test-data-v2`, etc.)
- If cache becomes corrupted or invalidated

**Updating Test Data:**
To regenerate test data with new parameters or scenarios, use the [PLINK2SYN](https://github.com/physioforecast/PLINK2SYN) pipeline and update the Google Drive archive.

### Expected Input Files

Tests expect data files in `test_data/` directory:

**For each dataset:**
- `{dataset}_recode_varIDs.pgen` - PLINK2 genotype file
- `{dataset}_recode_varIDs.pvar` - Variant information
- `{dataset}_recode_varIDs.psam` - Sample information
- `{dataset}_combined_phenocov.csv` - Phenotypes and covariates
- `{dataset}_recode_varIDs_subset_{n}.keep` - Sample subset files

**R Reference Files:**
- `{dataset}_{phenotype}_{covariate}_glm_{model}_keep{n}_{model}.csv`

**Local Testing:**
For local development, obtain test data separately or extract from cached source.

## Validation Criteria

Tests pass when all correlations between PLINK2 and R outputs meet the threshold:
- Beta coefficients correlation ≥ 0.8
- Standard error correlation ≥ 0.8
- P-value correlation ≥ 0.8

Threshold configurable via `correlation_threshold` in PLINK2_GLM_TEST_CONFIG.sh

## CI Integration

### GitHub Actions Workflow

Tests are automated via [.github/workflows/auto_checkcommits_tests_linuxmac.yml](/.github/workflows/auto_checkcommits_tests_linuxmac.yml)

**Trigger Schedule:**
- **Automatic:** Every Sunday at midnight (UTC)
- **Manual:** Via workflow_dispatch in GitHub Actions UI

**How It Works:**

1. **Commit Check:** Compares latest master commit with last successful workflow run
2. **Conditional Execution:** Only runs if new commits detected
3. **Matrix Strategy:** Tests run on both `ubuntu-latest` and `macos-latest`
4. **Batch Rotation:** Automatically cycles through batches 1-12 on each run
   - Uses cached `.batch_counter` file to track progress
   - Each workflow run tests the next batch in sequence
   - After batch 12, wraps back to batch 1
5. **Test Data:** Restores from GitHub Actions cache (populated by [cache_test_data.yml](/.github/workflows/cache_test_data.yml))
6. **Workflow Steps:**
   - Install system dependencies (LAPACK, OpenBLAS, etc.)
   - Restore cached test data (key: `test-data-v1`)
   - Build PLINK2 via DYNAMICBUILD_LINUXMAC_ROOT.sh
   - Run GLM tests for current batch via GLM_TESTS.sh
   - Cleanup via CLEANUP.sh

**Complete Test Coverage:**
- All 108 tests (12 batches) run every ~12 weeks
- Each Sunday runs 1 batch on both Linux and macOS
- 2 platforms × 9 tests per batch = 18 test executions per week

### Manual CI Testing

To test specific batches locally:
```bash
# Run specific batch
export GLM_BATCH_NUM=5
./2.0/build_dynamic/CI-SCRIPTS/GLM_TESTS.sh

# Or test all batches sequentially
for i in {1..12}; do
  export GLM_BATCH_NUM=$i
  ./2.0/build_dynamic/CI-SCRIPTS/GLM_TESTS.sh
done
```

## Troubleshooting

**Missing test data in CI:**
- Ensure [cache_test_data.yml](/.github/workflows/cache_test_data.yml) workflow has been run at least once
- Check that cache key matches (`test-data-v1`)
- Manually trigger cache_test_data.yml workflow if needed
- Verify Google Drive file is accessible

**Missing files error (local):**
- Ensure `test_data/` directory exists with required input files
- Check that R reference CSV files are present
- Contact maintainer for test data access

**Correlation failures:**
- Review PLINK2 vs R comparison output in results directory
- Check for numerical precision issues
- Verify input data integrity
- Ensure correct PLINK2 build and R reference files match

**Python errors:**
- Ensure pandas and numpy are installed (`pip install pandas numpy`)
- Check Python version compatibility (3.x required)
- Verify virtual environment is activated if using one

## Archive

The `archive/` directory contains deprecated scripts kept for reference:
- Old test implementations
- Development/debug scripts
- Superseded utilities

These are not actively maintained and should not be used for testing.

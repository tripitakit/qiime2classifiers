# QIIME2 Taxonomic Classifiers Repository

This repository contains pre-trained taxonomic classifiers for microbial community analysis using QIIME2.

#### SILVA 138.2 SSU NR99 341F-806R Classifier

- **File**: `silva-138.2-ssu-nr99-341f-806r-classifier.qza`
- **Database**: SILVA 138.2 SSU NR99
- **Target Region**: 341F-806R (V3-V4)
- **QIIME2 Version**: 2025.4
- **Training Method**: RESCRIPt pipeline with Naive Bayes classifier

This is the primary classifier for V3-V4 amplicon sequencing using the 341F/806R primer pair. It was built using the QIIME2 RESCRIPt plugin following current best practices for reference database processing.

**Primer Sequences:**

- Forward (341F): `CCTACGGGNGGCWGCAG`
- Reverse (806R): `GGACTACNVGGGTWTCTAAT`

### Evaluation Artifacts

#### SILVA 138.2 341F-806R Classifier Evaluation

- **Fit Classifier Evaluation**: `silva-138.2-ssu-nr99-341f-806r-taxonomy-fit-classifier-evaluation.qzv`

- **Taxonomy Evaluation**: `silva-138.2-ssu-nr99-341f-806r-taxonomy-evaluation.qzv`

## SILVA 138.2 Classifier Training Protocol

The SILVA 138.2 341F-806R classifier was built using the following RESCRIPt pipeline commands:

### 1. Environment Setup

```bash
conda activate qiime2-amplicon-2025.4
```

### 2. Download SILVA Database

```bash
qiime rescript get-silva-data \
    --p-version '138.2' \
    --p-target 'SSURef_NR99' \
    --o-silva-sequences silva-138.2-ssu-nr99-rna-seqs.qza \
    --o-silva-taxonomy silva-138.2-ssu-nr99-tax.qza
```

### 3. Convert RNA to DNA Sequences

```bash
qiime rescript reverse-transcribe \
    --i-rna-sequences silva-138.2-ssu-nr99-rna-seqs.qza \
    --o-dna-sequences silva-138.2-ssu-nr99-seqs.qza
```

### 4. Quality Control - Cull Low-Quality Sequences

```bash
qiime rescript cull-seqs \
    --i-sequences silva-138.2-ssu-nr99-seqs.qza \
    --o-clean-sequences silva-138.2-ssu-nr99-seqs-cleaned.qza \
    --p-n-jobs 12
```

### 5. Filter Sequences by Length and Taxonomy

```bash
qiime rescript filter-seqs-length-by-taxon \
    --i-sequences silva-138.2-ssu-nr99-seqs-cleaned.qza \
    --i-taxonomy silva-138.2-ssu-nr99-tax.qza \
    --p-labels Archaea Bacteria Eukaryota \
    --p-min-lens 900 1200 1400 \
    --o-filtered-seqs silva-138.2-ssu-nr99-seqs-filt.qza \
    --o-discarded-seqs silva-138.2-ssu-nr99-seqs-discard.qza
```

### 6. Dereplicate Sequences (Unique Mode)

```bash
qiime rescript dereplicate \
    --i-sequences silva-138.2-ssu-nr99-seqs-filt.qza \
    --i-taxa silva-138.2-ssu-nr99-tax.qza \
    --p-mode 'uniq' \
    --o-dereplicated-sequences silva-138.2-ssu-nr99-seqs-derep-uniq.qza \
    --o-dereplicated-taxa silva-138.2-ssu-nr99-tax-derep-uniq.qza \
    --p-threads 12
```

### 7. Extract Amplicon Region (341F-806R)

```bash
qiime feature-classifier extract-reads \
    --i-sequences silva-138.2-ssu-nr99-seqs-derep-uniq.qza \
    --p-f-primer CCTACGGGNGGCWGCAG \
    --p-r-primer GGACTACNVGGGTWTCTAAT \
    --p-n-jobs 12 \
    --p-read-orientation 'forward' \
    --o-reads silva-138.2-ssu-nr99-seqs-341f-806r.qza
```

### 8. Final Dereplication of Amplicon Sequences

```bash
qiime rescript dereplicate \
    --i-sequences silva-138.2-ssu-nr99-seqs-341f-806r.qza \
    --i-taxa silva-138.2-ssu-nr99-tax-derep-uniq.qza \
    --p-mode 'uniq' \
    --o-dereplicated-sequences silva-138.2-ssu-nr99-seqs-341f-806r-uniq.qza \
    --o-dereplicated-taxa silva-138.2-ssu-nr99-tax-341f-806r-derep-uniq.qza \
    --p-threads 12
```

### 9. Build and Evaluate Classifier

```bash
qiime rescript evaluate-fit-classifier \
    --i-sequences silva-138.2-ssu-nr99-seqs-341f-806r-uniq.qza \
    --i-taxonomy silva-138.2-ssu-nr99-tax-341f-806r-derep-uniq.qza \
    --o-classifier silva-138.2-ssu-nr99-341f-806r-classifier.qza \
    --o-observed-taxonomy silva-138.2-ssu-nr99-341f-806r-taxonomy-predicted-taxonomy.qza \
    --o-evaluation silva-138.2-ssu-nr99-341f-806r-taxonomy-fit-classifier-evaluation.qzv \
    --p-n-jobs 12
```

### 10. Evaluate Taxonomic Performance

```bash
qiime rescript evaluate-taxonomy \
    --i-taxonomies silva-138.2-ssu-nr99-tax-341f-806r-derep-uniq.qza silva-138.2-ssu-nr99-341f-806r-taxonomy-predicted-taxonomy.qza \
    --p-labels silva-138.2-ssu-nr99-341f-806r-taxonomy silva-138.2-ssu-nr99-341f-806r-predicted-taxonomy \
    --o-taxonomy-stats silva-138.2-ssu-nr99-341f-806r-taxonomy-evaluation.qzv
```

## Viewing Evaluation Results

To view the classifier evaluation visualizations:

```bash
# View fit classifier evaluation
qiime tools view silva-138.2-ssu-nr99-341f-806r-taxonomy-fit-classifier-evaluation.qzv

# View taxonomy evaluation
qiime tools view silva-138.2-ssu-nr99-341f-806r-taxonomy-evaluation.qzv
```

## References

- [SILVA Database](https://www.arb-silva.de/)
- [QIIME2 RESCRIPt Plugin](https://github.com/bokulich-lab/RESCRIPt)
- [RESCRIPt Tutorial](https://forum.qiime2.org/t/processing-filtering-and-evaluating-the-silva-database-and-other-reference-sequence-data-with-rescript/15494)

## License

Classifiers are provided as-is for research purposes. Please cite the appropriate database sources when using these classifiers in publications.

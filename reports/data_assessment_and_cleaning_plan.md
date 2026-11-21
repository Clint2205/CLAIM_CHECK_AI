# ClaimCheck-AI: Data Assessment Findings and Cleaning Plan

**Project:** ClaimCheck-AI  
**Dataset:** FEVER v1.0  
**Assessment stage:** Data Understanding and Dataset Assessment  
**Status:** Assessment completed; data cleaning ready to begin

---

## 1. Purpose

This report documents the initial data understanding, statistical assessment, data-quality investigation and proposed cleaning strategy for the FEVER v1.0 dataset used in ClaimCheck-AI.

The purpose of this assessment is to establish the condition and characteristics of the original data before applying transformations or preparing it for NLP modelling.

The original dataset files must remain unchanged. All future transformations will be performed on separate working copies.

## 2. Dataset overview

Two labelled dataset files were acquired:

| Dataset | Records |
|---|---:|
| Training | 145,449 |
| Shared Task Development | 19,998 |
| **Total** | **165,447** |

The datasets contain five fields:

- `id`: Unique record identifier.
- `verifiable`: Indicates whether the claim is verifiable within the task.
- `label`: The classification target.
- `claim`: The natural-language claim.
- `evidence`: Nested evidence annotations.

The three classification labels are:

- `SUPPORTS`
- `REFUTES`
- `NOT ENOUGH INFO`

The evidence field contains nested lists. Evidence items include annotation identifiers and, where available, Wikipedia page titles and sentence indices.

## 3. Descriptive statistical findings

### 3.1 Training label distribution

| Label | Count | Percentage |
|---|---:|---:|
| SUPPORTS | 80,035 | 55.03% |
| NOT ENOUGH INFO | 35,639 | 24.50% |
| REFUTES | 29,775 | 20.47% |
| **Total** | **145,449** | **100%** |

The training dataset is imbalanced, with SUPPORTS representing approximately 55% of records.

This imbalance may influence model training and evaluation. Accuracy alone may not adequately describe performance across all three classes.

### 3.2 Development label distribution

| Label | Count | Percentage |
|---|---:|---:|
| SUPPORTS | 6,666 | 33.33% |
| NOT ENOUGH INFO | 6,666 | 33.33% |
| REFUTES | 6,666 | 33.33% |
| **Total** | **19,998** | **100%** |

The development dataset is evenly distributed across the three classes.

This makes it useful for comparing performance across classes, but its distribution differs from the training dataset. Development metrics should therefore be interpreted with this difference in mind.

## 4. Confidence interval analysis

Wilson 95% confidence intervals were calculated for the training label proportions.

| Label | Proportion | Lower bound | Upper bound |
|---|---:|---:|---:|
| SUPPORTS | 55.03% | 54.77% | 55.28% |
| NOT ENOUGH INFO | 24.50% | 24.28% | 24.72% |
| REFUTES | 20.47% | 20.26% | 20.68% |

The intervals are relatively narrow because of the large number of training records.

**Interpretation and limitation:** The observed proportions describe the actual training dataset. The confidence intervals quantify uncertainty under a binomial sampling model. FEVER is a constructed benchmark rather than a random sample of all real-world claims. These intervals must not be interpreted as estimates of the prevalence of true, false or unverifiable claims in the general population.

## 5. Training and development distribution comparison

A chi-square test of independence was performed to investigate the association between dataset split and label distribution.

| Statistic | Result |
|---|---:|
| Chi-square statistic | 3435.3751 |
| Degrees of freedom | 2 |
| P-value | Extremely small; displayed as 0.0 by the initial output |
| Cramér's V | 0.1019 |
| Total observations | 165,447 |

The test provides strong evidence that the label distributions differ between the training and development datasets.

Cramér's V indicates a modest association in this comparison. The very small p-value and relatively modest effect size are not contradictory: a large sample can make comparatively small differences statistically detectable.

The test assumes independent observations. Repeated claims within the datasets may affect this assumption, so the result is treated as exploratory rather than definitive.

**Research implication:** Model evaluation should include macro-F1, per-class precision, per-class recall and a confusion matrix, alongside accuracy. Evaluation results should also be interpreted in the context of the development dataset's balanced class distribution.

## 6. Duplicate claim investigation

### 6.1 Training dataset

| Finding | Result |
|---|---:|
| Rows involved in repeated claims | 15,798 |
| Unique repeated claim texts | 5,894 |
| Repeated claims with identical labels | 5,141 |
| Repeated claims with different labels | 753 |
| Repeated claims with different evidence annotations | 5,894 |
| Repeated claims with different IDs | 5,894 |

### 6.2 Development dataset

| Finding | Result |
|---|---:|
| Rows involved in repeated claims | 736 |
| Unique repeated claim texts | 335 |
| Repeated claims with identical labels | 281 |
| Repeated claims with different labels | 54 |
| Repeated claims with different evidence annotations | 335 |
| Repeated claims with different IDs | 335 |

Repeated wording does not automatically mean that records are redundant. Different evidence annotations may represent distinct annotation decisions or evidence relationships.

Claims with conflicting labels require further investigation. They must not automatically be classified as incorrect annotations.

A preliminary evidence-normalisation method compared claim text, label and unique page/sentence pairs. It produced 6,337 duplicate rows under that particular comparison rule. This is a preliminary metric, not a confirmed count of genuinely redundant records, because the method flattened nested evidence groups and ignored incomplete evidence fields.

The original evidence structure must be retained until a more rigorous comparison method is established.

## 7. Cross-dataset overlap investigation

The initial comparison found:

| Check | Result |
|---|---:|
| Overlapping IDs | 0 |
| Identical claim texts in both datasets | 3 |

The three overlapping claims were:

1. Floyd Mayweather Jr. is a professional boxer.
2. Saturn is a planet in the Solar System.
3. Warren Beatty co-wrote Reds.

All three had the same label in both datasets, but their evidence annotations differed.

These records are potential evaluation leakage cases because identical claim wording occurs in both splits. The original records will be preserved while their implications are investigated.

## 8. Evidence structure and availability assessment

The evidence field was investigated separately from ordinary column-level missingness.

### 8.1 Training evidence assessment

| Label | Claims | Evidence groups | Annotation items | Usable evidence items | Incomplete evidence items |
|---|---:|---:|---:|---:|---:|
| NOT ENOUGH INFO | 35,639 | 35,639 | 47,609 | 0 | 47,609 |
| REFUTES | 29,775 | 60,227 | 70,066 | 70,066 | 0 |
| SUPPORTS | 80,035 | 161,249 | 193,756 | 193,756 | 0 |

### 8.2 Development evidence assessment

| Label | Claims | Evidence groups | Annotation items | Usable evidence items | Incomplete evidence items |
|---|---:|---:|---:|---:|---:|
| NOT ENOUGH INFO | 6,666 | 6,666 | 8,941 | 0 | 8,941 |
| REFUTES | 6,666 | 12,227 | 14,017 | 14,017 | 0 |
| SUPPORTS | 6,666 | 12,498 | 14,608 | 14,608 | 0 |

Both datasets showed zero malformed evidence items and zero missing annotator IDs.

For SUPPORTS and REFUTES, all inspected evidence items had identifiable page titles and sentence indices.

For NOT ENOUGH INFO, the annotation structures existed, but the evidence items did not contain identifiable page titles or sentence indices. This is consistent with the expected FEVER representation for claims without sufficient annotated evidence.

**Important distinction:** A record can contain a valid annotation structure without containing an identifiable evidence sentence. The presence of a nested list must not be treated as proof that usable evidence exists.

The incomplete evidence fields in NOT ENOUGH INFO must not automatically be treated as data corruption or replaced with fabricated evidence.

## 9. Baseline data integrity assessment

The initial baseline checks were performed before data cleaning.

| Check | Training | Development |
|---|---:|---:|
| Total records | 145,449 | 19,998 |
| Duplicate IDs | 0 | 0 |
| Missing IDs | 0 | 0 |
| Missing claims | 0 | 0 |
| Empty claims | 0 | 0 |
| Missing labels | 0 | 0 |
| Invalid labels | 0 | 0 |
| Missing evidence columns | 0 | 0 |

The three expected labels were present, and the initial checks found no missing or empty claim text, missing IDs, duplicate IDs or missing top-level evidence columns.

This establishes a useful baseline against which future cleaning operations can be validated.

It does not establish that every annotation is semantically correct or that all evidence relationships are free from issues.

## 10. Data cleaning plan and justification

The following decisions will guide the cleaning stage.

| Issue | Planned action | Justification |
|---|---|---|
| Class imbalance | Preserve all three classes | Avoid losing information or changing the original class distribution without a modelling justification. |
| NOT ENOUGH INFO evidence placeholders | Preserve original nested annotations | These are expected annotation representations, not automatically defective records. |
| Missing evidence metadata | Investigate by field and label | Distinguish expected missingness from genuine structural problems. |
| Repeated claim texts | Retain initially and flag | Identical wording does not prove identical evidence or annotation meaning. |
| Conflicting labels | Flag for manual investigation | Avoid making unsupported assumptions about annotation errors. |
| Nested evidence groups | Preserve grouping and ordering | Flattening may discard meaningful evidence relationships. |
| Duplicate IDs | Validate uniqueness | IDs should identify individual records. |
| Three overlapping train/development claims | Preserve and document | Investigate possible evaluation leakage before deciding on a treatment. |
| Missing or empty claims | Flag for investigation | Claims are essential input to the verification task. |
| Invalid labels | Flag for investigation | Only the three expected classification labels should be accepted. |

### 10.1 Proposed data layers

The project will maintain three distinct data layers:

**Raw**
- Original downloaded FEVER JSONL files.
- Read-only during the project.
- Never overwritten by cleaning operations.

**Interim**
- Statistical summaries.
- Duplicate audits.
- Evidence structure audits.
- Validation reports.
- Intermediate analytical outputs.

**Processed**
- Validated and cleaned working datasets.
- Explicitly documented transformations.
- Preserved record IDs and label relationships.
- Suitable for downstream modelling once validation is complete.

### 10.2 Planned cleaning sequence

1. Establish and preserve baseline integrity results.
2. Inspect nested evidence structures and missingness patterns.
3. Validate record IDs, claim text and classification labels.
4. Develop a structure-aware evidence normalisation method without overwriting original evidence.
5. Investigate repeated claims and conflicting labels.
6. Investigate cross-split claim overlap and potential leakage.
7. Apply only justified transformations to working copies.
8. Re-run validation checks after each transformation.
9. Save processed datasets and a cleaning log.
10. Compare the processed datasets against the original baseline.

## 11. Reproducibility and data governance

All transformations must be reproducible and justified.

The project will:

- Preserve the original downloaded data.
- Record the reason for every transformation.
- Keep record IDs available for traceability.
- Avoid silently deleting records.
- Preserve evidence relationships unless a documented transformation requires otherwise.
- Record before-and-after row counts.
- Validate class distributions after cleaning.
- Maintain separate training and development datasets.
- Document unresolved issues and limitations.

The cleaning process should not introduce information from the development dataset into model training.

## 12. Outstanding investigations

The following issues remain open:

1. Determine whether repeated claim annotations with conflicting labels represent genuine annotation disagreements or different interpretations.
2. Establish a structure-aware definition of equivalent evidence annotations.
3. Investigate the three overlapping claim texts across training and development.
4. Consider the effect of repeated claims on statistical test independence.
5. Verify FEVER source attribution and licensing requirements before redistributing any dataset-derived material.

## 13. Conclusion

The initial FEVER assessment found that both datasets have valid top-level structures, unique IDs, populated claim and label fields, and no malformed evidence items under the checks performed.

The principal considerations for data cleaning are class imbalance, nested evidence annotations, repeated claims, conflicting labels and potential cross-split overlap.

No records have been removed or modified as a result of this assessment.

The next stage will implement a controlled, reproducible cleaning process while preserving the original FEVER data and maintaining a clear audit trail.

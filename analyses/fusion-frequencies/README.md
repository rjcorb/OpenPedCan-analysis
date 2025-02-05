## Annotate fusion tables with fusion frequencies and fused gene frequencies

Adapted from [snv-frequencies](https://github.com/logstar/OpenPedCan-analysis/tree/snv-freq/analyses/snv-frequencies) authored by Yuanchao Zhang ([@logstar](https://github.com/logstar))

**Module authors:** Krutika Gaonkar ([@kgaonkar6](https://github.com/kgaonkar6)) and Jo Lynne Rokita (@jharenza)[https://github.com/jharenza]

### Purpose
Uses `fusion-putative-oncogenic.tsv` and `fusion-dgd.tsv.gz`, a `Alt_ID` for each `FusionName` and `Fusion_Type` concatenated by `"_"` OR Fused Gene to count occurence in  each `cancer_group` in a given dataset, primary or relapse cohorts.

Note: DGD cohort is fusion panel data and is currently not included in "All cohorts" calculations.

Each gene invloved in the fusion is annotated by a Gene_Position, for example:
 - In a genic fusion where both breakpoints are within gene body the Gene_Position `Gene1A` will be the 5' gene and Gene1B will be 3' gene
 - In an intergenic fusion where one or both breakpoint are outside the gene body, if the 5' breakpoint is a region between GeneX and GeneY  the Gene_Position of GeneX will be Gene1A, for GeneY will be Gene2A and their 3' breakpoint is within GeneE will be denoted as Gene_Position `Gene1B`.
 - In an intergenic fusion where one or both breakpoint are outside the gene body, if the 3' breakpoint is within GeneE the Gene_Position of GeneE will be Gene1A and the 5' breakpoint is within GeneY and GeneZ, Gene Y will be denoted as Gene_Position `Gene1B` and GeneZ will be denoted as `Gene2B`.   


#### Frequency annotation

For each `cancer_group` and `cohort`(s) combination is considered a `cancer_group_cohort`. `cancer_group_cohort` with `n_samples` >= 3,  `Frequency_in_overall_dataset`, `Frequency_in_primary_tumors`, and `Frequency_in_relapse_tumors` were computed. 
In this module, we generated two fusion frequency tables - one `putative-oncogene-fusion-freq.jsonl.gz` calculate the frequency of a specific fusion in primary tumors, relapse tumors, and overall dataset. 
In `putative-oncogene-fused-gene-freq.jsonl.gz` on the other hand, we calculate the frequency of a particular gene being fused in primary tumors, relapse tumors, and overall dataset.
As long as a gene is a partner in a particular fusion (from 1A, 1B, 2A or 2B), it is counted as fused in that particular sample. 

Table one - `putative-oncogene-fusion-freq.jsonl.gz`
- `Frequency_in_overall_dataset`:
  - For each unique fusion, count the number of patients (identified by `Kids_First_Participant_ID`) that have the fusion, and call this number `Total_alterations`.
- Count the total number of patients in the `cancer_group_cohort`, and call this number `Patients_in_dataset`.
- `Frequency_in_overall_dataset = Total_alterations / Patients_in_dataset`.

Please **NOTE** that `Total_alterations` is the number of patients that have the particular fusion - NOT the total number of fusions. 
If a specific fusion occured 3 times but they were all from the same patient, the `Total_alterations` would equal to 1 not 3.
Therefore, `Total_alterations` cannot be directly summed in most cases, because the value corresponds to the size of a set of patients, and union of the patient sets need to be taken to have a grand "sum" of Total_alterations. For example,
- Row 1 altered patients are {a, b, c}; Total_alterations = 3.
- Row 2 altered patients are {a, c, d}; Total_alterations = 3.
- Row 1 and 2 altered patients should be {a, b, c, d}; Total_alterations = 4.

Please also **NOTE** that just like `Total_alterations`, `Total_relapse_tumors_alterated` and `Total_primary_tumors_alterated` are also the number of patients that have the particular fusion - NOT the total number of fusions. 

- `Frequency_in_primary_tumors`:

  -  For any `cancer_group_cohort` that contains `cancer_group` that has **multiple cohorts** listed in the `cohort` column (in the intermediate table `nf_cancer_group_cohort_summary_df`), independent sample list for all cohorts is used and the calculation is done as followed:
    - For each unique variant, count the number of samples (identified by `Kids_First_Biospecimen_ID`) that are in the `../../data/independent-specimens.rnaseq.primary.tsv`, and call this number `Total_primary_tumors_alterated`.
    - Count the total number of samples in the `cancer_group_cohort` that are also in the `.../../data/independent-specimens.rnaseq.primary.tsv`, and call this number `Primary_tumors_in_dataset`.
    - `Frequency_in_primary_tumors = Total_primary_tumors_alterated / Primary_tumors_in_dataset`.
    
  -  For any `cancer_group_cohort` that contains `cancer_group` that maps to **single cohort** listed in the `cohort` column (in the intermediate table `nf_cancer_group_cohort_summary_df`), independent sample list for each cohort is used:
    - For each unique variant, count the number of samples (identified by `Kids_First_Biospecimen_ID`) that are in the `../../data/independent-specimens.rnaseq.primary.eachcohort.tsv`, and call this number `Total_primary_tumors_alterated`.
    - Count the total number of samples in the `cancer_group_cohort` that are also in the `.../../data/independent-specimens.rnaseq.primary.eachcohort.tsv`, and call this number `Primary_tumors_in_dataset`.
    - `Frequency_in_primary_tumors = Total_primary_tumors_alterated / Primary_tumors_in_dataset`.

- `Frequency_in_relapse_tumors`:

   -  For any `cancer_group_cohort` that contains `cancer_group` that has **multiple cohorts** listed in the `cohort` column (in the intermediate table `nf_cancer_group_cohort_summary_df`), independent sample list for all cohorts is used and the calculation is done as followed:
    - For each unique variant, count the number of samples (identified by `Kids_First_Biospecimen_ID`) that are in the `../../data/independent-specimens.rnaseq.relapase.eachcohort.tsv`, and call this number `Total_relapse_tumors_alterated`.
    - Count the total number of samples in the `cancer_group_cohort` that are also in the `.../../data/independent-specimens.rnaseq.relapase.eachcohort.tsv`, and call this number `Relapse_tumors_in_dataset`.
    - `Frequency_in_relapse_tumors = Total_relapse_tumors_alterated / Relapse_tumors_in_dataset`.
  
  -  For any `cancer_group_cohort` that contains `cancer_group` that maps to **single cohort** listed in the `cohort` column (in the intermediate table `nf_cancer_group_cohort_summary_df`), independent sample list for each cohort is used:
    - For each unique variant, count the number of samples (identified by `Kids_First_Biospecimen_ID`) that are in the `../../data/independent-specimens.rnaseq.relapase.eachcohort.tsv`, and call this number `Total_relapse_tumors_alterated`.
    - Count the total number of samples in the `cancer_group_cohort` that are also in the `.../../data/independent-specimens.rnaseq.relapase.eachcohort.tsv`, and call this number `Relapse_tumors_in_dataset`.
    - `Frequency_in_relapse_tumors = Total_relapse_tumors_alterated / Relapse_tumors_in_dataset`.
  
Format the SNV mutation frequency table according to the latest spreadsheet that is attached in <https://github.com/PediatricOpenTargets/ticket-tracker/issues/64>.

The fusion frequency tables of all `cancer_group_cohort`s were merged.

Additionallly, some field names were changed for OT compatbiitliy:
- `Gene_Ensembl_Id` is changed to `targetFromSourceId`
- `EFO` to `diseaseFromSourceMappedId`
Three additional columns added: 
- `chop_uuid` includes 32 digit UUIDs that are unique for each row of the file 
- `datatypeId` column with `somatic_mutation` in each row
- `datasourceId` column with `chop_putative_oncogene_fusion` in each row

The combined table was written out as `putative-oncogene-fusion-freq.jsonl.gz`.
Table fields for this table include: `FusionName`, `Fusion_Type`, `Gene_symbol`, `Gene_Position`, `Fusion_anno`, `BreakpointLocation`,`annots`, `Kinase_domain_retained_Gene1A`, `Kinase_domain_retained_Gene1B`, `Reciprocal_exists_either_gene_kinase`, `Gene1A_anno`, `Gene1B_anno`, `Gene2A_anno`, `Gene2B_anno`, `targetFromSourceId`, `Disease`, `MONDO`, `PMTL`, `diseaseFromSourceMappedId`, `Dataset`, `Total_alterations_Over_Patients_in_dataset`, `Frequency_in_overall_dataset`, `Total_primary_tumors_mutated_Over_Primary_tumors_in_dataset`, `Frequency_in_primary_tumors`, `Total_relapse_tumors_mutated_Over_Relapse_tumors_in_dataset`, `Frequency_in_relapse_tumors`, `chop_uuid`, `datatypeId` and `datasourceId`.

Table two - `putative-oncogene-fused-gene-freq.jsonl.gz`
The overall calculation logic is identical to table one. 
The difference is instead of calculating the frequency for each unique fusion, we are calculating the frequency for each unique gene that is present within a fusion (from 1A, 1B, 2A or 2B in a particular fusion) in different tumor types.
After merging fusion frequency tables of all `cancer_group_cohort`s, some field names were changed for OT compatbiitliy:
- `Gene_Ensembl_Id` is changed to `targetFromSourceId`
- `EFO` to `diseaseFromSourceMappedId`
Three additional columns added: 
- `chop_uuid` includes 32 digit UUIDs that are unique for each row of the file 
- `datatypeId` column with `somatic_mutation` in each row
- `datasourceId` column with `chop_putative_oncogene_fused_gene` in each row

the combined table was written out as `putative-oncogene-fused-gene-freq.jsonl.gz`.
Table fields for this table include:
`Gene_symbol`,`targetFromSourceId`, `Disease`, `MONDO`, `PMTL`, `diseaseFromSourceMappedId`, `Dataset`, `Total_alterations_Over_Patients_in_dataset`, `Frequency_in_overall_dataset`, `Total_primary_tumors_mutated_Over_Primary_tumors_in_dataset`, `Frequency_in_primary_tumors`, `Total_relapse_tumors_mutated_Over_Relapse_tumors_in_dataset`, `Frequency_in_relapse_tumors`, `chop_uuid`, `datatypeId` and `datasourceId`.

#### Additional annotation

[long-format-table-utils](https://github.com/PediatricOpenTargets/OpenPedCan-analysis/tree/dev/analyses/long-format-table-utils) provides the annotation for columns  EFO, MONDO, PMTL and Gene_full_name.

### Analysis scripts

### `00-annotate-panel-fusions.Rmd`
This script annotates DGD panel fusions and genes using the ` annotate_fusion_calls` function, adapted from _annoFuse_.

### `01-fusion-frequencies.R`
This script calculates fusion and fused gene frequencies for each `cancer_group` and `cohort`, primary and relapse.
For the above `cancer_group`, this script also calculates fusion and fused gene frequencies for `All cohorts`, except for `CHOP P30 Panel` (DGD). 

Usage:

```bash
bash run-frequencies.sh
```

Input:

- `../../data/histologies.tsv`
- `../../data/fusion-putative-oncogenic.tsv`
- `input/fusion-dgd-tiers-FusionAnnotator.tsv`
- `../../data/independent-specimens.rnaseq.primary.eachcohort.tsv`
- `../../data/independent-specimens.rnaseq.relapse.eachcohort.tsv`
- `../../data/independent-specimens.rnaseq.primary.tsv`
- `../../data/independent-specimens.rnaseq.relapse.tsv`

```
results/
├── fusion-dgd-annotated.tsv
├── putative-oncogene-fused-gene-freq.json.gz
├── putative-oncogene-fused-gene-freq.jsonl.gz
├── putative-oncogene-fused-gene-freq.tsv.gz
├── putative-oncogene-fusion-freq.json.gz
├── putative-oncogene-fusion-freq.jsonl.gz
└── putative-oncogene-fusion-freq.tsv.gz
```

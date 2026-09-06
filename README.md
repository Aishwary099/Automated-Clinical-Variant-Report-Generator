# Automated-Clinical-Variant-Report-Generator
Readme · MD
Automated Clinical Variant Report Generator

A Python tool that turns raw somatic variant-calling output (VCF + ANNOVAR annotation) into a clinical-style variant report — automatically — in Excel, PDF, and Word formats. Built to mirror the kind of report a cancer genomics analyst would hand to an oncologist: which mutations were found, how confident we are, which gene/protein they affect, and what that might mean for treatment.

Why this exists

Manually formatting variant tables into a shareable, readable report is repetitive and error-prone. This tool automates that last mile: point it at your variant-calling pipeline's output, and it produces a ready-to-read report with ACMG/CancerVar classification, gene/transcript/protein details, and PharmGKB-style drug-response notes — with pathogenic findings color-highlighted so nothing gets missed.

What it produces
Format	What's in it
.xlsx	Full variant table, 22 columns, one row per variant, pathogenic/likely-pathogenic rows color-highlighted
.pdf	One-page clinical summary: patient info, pathogenic findings table, gene-level drug-response notes
.docx	Same content as the PDF, in an editable Word document

See sample_output/ for real generated examples (variant_report.xlsx, .pdf, .docx) you can open right now.

Screenshots
PDF report	Word report
Show Image	Show Image
Excel column reference
Column	Source
Category	Somatic/Germline label
Chrom, Position, Ref, Variant	VCF
Zygosity	VCF genotype (GT)
Frequency	Variant allele frequency, computed from VCF AD field
Quality	VCF QUAL
Original / Downsample Coverage & Allele Cov	VCF DP/AD vs. optional downsampled DDP/DAD fields
Gene Symbol, Transcript ID, Exon No, cDNA/Protein Change	ANNOVAR refGene annotation
COSMIC ID	ANNOVAR + COSMIC database
Variant Category	SNV / Insertion / Deletion, derived from REF/ALT length
Inferred Activity	Gene functional-impact lookup table (extend with OncoKB API for production use)
Variant Classification	ACMG (InterVar) or cancer-tier (CancerVar) classification
dbSNP ID, 1000 genomes Frequency, gnomAD Frequency	ANNOVAR avsnp150 / 1000g / gnomAD databases
Quick start
bash
git clone https://github.com/Aishwary099/clinical-variant-report-generator.git
cd clinical-variant-report-generator
pip install -r requirements.txt

python3 src/generate_report.py \
  --vcf sample_data/sample.vcf \
  --multianno sample_data/sample.hg38_multianno.txt \
  --classification sample_data/sample.intervar.tsv \
  --tumor-sample TUMOR --normal-sample NORMAL \
  --patient-id "Demo_Patient_001" \
  --formats xlsx,pdf,docx \
  --out sample_output/variant_report

That's it — sample_output/variant_report.{xlsx,pdf,docx} are generated. Try it on the bundled sample data first (real well-known cancer hotspots — TP53 R175H, BRAF V600E, KRAS G12C, BRCA1 frameshift — used to demonstrate the tool), then point --vcf/--multianno/--classification at your own pipeline's real output.

Input file requirements
Flag	Required	What it is	How to produce it
--vcf	Yes	Somatic VCF with tumor (+optional normal) sample columns, carrying GT, DP, AD FORMAT fields	GATK Mutect2, bcftools, or any standard variant caller
--multianno	Yes	ANNOVAR *_multianno.txt output	table_annovar.pl with refGene,avsnp150,clinvar,1000g,gnomad protocols
--classification	No	4-column + classification TSV (Chr,Start,Ref,Alt,Classification)	Run InterVar/CancerVar, then see docs/extract_classification.py to convert their native output into this simple format
--gene-activity	No	2-column TSV (gene, activity) to extend the built-in functional-impact table	Your own curation, or an OncoKB API export
--drug-response	No	2-column TSV (gene, note) to extend the built-in PharmGKB-style notes	A downloaded PharmGKB clinical-annotations export
Project structure
clinical-variant-report-generator/
├── src/
│   └── generate_report.py       # the tool
├── sample_data/                 # example VCF + ANNOVAR + classification input
├── sample_output/                # example generated .xlsx / .pdf / .docx
├── docs/
│   └── extract_classification.py # converts real InterVar/CancerVar output to the simple format above
├── requirements.txt
├── LICENSE
└── README.md
Extending this project
Inferred Activity: currently a small built-in lookup table for common cancer genes. For a clinically rigorous version, query the OncoKB API (free academic token) instead.
Drug-Response Notes: currently a small built-in table. Swap in a downloaded PharmGKB clinical-annotations export, joined on gene symbol, for broader coverage.
COSMIC ID / ClinVar / dbSNP / 1000G / gnomAD: come straight from ANNOVAR — see the companion end-to-end pipeline project for the full BWA → GATK → ANNOVAR pipeline that produces the --vcf and --multianno inputs this tool consumes.
Disclaimer

Built as a learning/portfolio project using public reference data (not real patient data). Not validated for clinical use.

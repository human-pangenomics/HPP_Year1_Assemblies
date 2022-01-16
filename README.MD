## Year 1 Assemblies

![White Logo](https://s3-us-west-2.amazonaws.com/human-pangenomics/backup/logo-proof-full.png)

*This repo describes assemblies produced by the [Human Pangenome Reference Consortium](https://humanpangenome.org/) from year 1 data. Assemblies for 47 samples are available. For information about data reuse and publicating with HPRC data please see the HPRC's [Data Use Protocol](https://humanpangenome.org/data-use-protocol/).*

------------------

### Obtaining assemblies

All assemblies are accessioned at GenBank. They can be downloaded from [public
HPRC S3 bucket][s3-pub] with no egress fee. Individual S3 and GCP URLs to each
assembly can be found in the [index
file](assembly_index/Year1_assemblies_v2_genbank.index) in this repo.

Assemblies are also [available][yr1-agc] as a single 1.5GB file in the
[AGC][agc] format. Users need to [download AGC][agc-dl] to extract individual
assemblies. For example:

```sh
# download all assemblies
curl -o HPRC-yr1.agc https://zenodo.org/record/5826274/files/HPRC-yr1.agc?download=1

# download precompiled AGC binary for Linux
curl -L https://github.com/refresh-bio/agc/releases/download/v1.1/agc-1.1_x64-linux.tar.gz|tar -zxvf - agc-1.1_x64-linux/agc

# list all samples
agc-1.1_x64-linux/agc listset HPRC-yr1.agc

# extract sample NA18906.1
agc-1.1_x64-linux/agc getset HPRC-yr1.agc NA18906.1 > NA18906.1.fa
```

[agc]: https://github.com/refresh-bio/agc
[agc-dl]: https://github.com/refresh-bio/agc/releases
[yr1-agc]: https://zenodo.org/record/5826274
[s3-pub]: https://s3-us-west-2.amazonaws.com/human-pangenomics/index.html?prefix=working/

<!--
**Note: The assemblies contained in the HPRC [S3](https://s3-us-west-2.amazonaws.com/human-pangenomics/index.html?prefix=working/) and [GCP](https://console.cloud.google.com/storage/browser/fc-4310e737-a388-4a10-8c9e-babe06aaf0cf/working?authuser=0) buckets referred to in this repo have not been fully QC'd, are not published, and many have known issues.**
-->

------------------

### Genbank Version Of Assemblies

Freeze 1 (v2) assemblies were uploaded to Genbank/NCBI and are now available. These assemblies are expected to be the final release of the year 1 assemblies and should be used for all analysis and pangenome work. A list of the current assemblies and their download links can be found in the index file [assembly_index/Year1_assemblies_v2_genbank.index](assembly_index/Year1_assemblies_v2_genbank.index).

As part of the upload to Genbank, contaminated contigs are identified and dropped. Some contigs which are almost certainly contamination were not identified, however. In addition, leading/trailing hard-masked nucleotides were trimmed from the contigs -- resulting in a co-ordinate change for those contigs. Lastly, contigs were renamed by Genbank with their accession IDs. Files have been provided in the genbank_changes folder to document these changes.

**Files in the genbank_changes folder:**
* y1_genbank_contig_translation_table.txt: Translation table for contig names
* y1_genbank_dropped_contigs.txt: Contigs dropped by Genbank 
* y1_genbank_remaining_potential_contamination.txt: Contigs that may be contamination
* y1_genbank_trimmed_sequence.txt: Sequence trimmed by Genbank

------------------

### Assembly Process

Assemblies were processed in AnVIL using publicly available workflows in the Human Pangenomics [hpp_production_workflows repo](https://github.com/human-pangenomics/hpp_production_workflows)

**A summary of the process is below:**
* Filter out sequences with adapters from HiFi data using [HiFiAdapterFilt](https://github.com/sheinasim/HiFiAdapterFilt)
* Run [hifiasm v0.14](https://github.com/chhylp123/hifiasm)
  * For information about the input HiFi and parent Illumina data see the [HPP Year 1 Data Freeze Repo](https://github.com/human-pangenomics/HPP_Year1_Data_Freeze_v1.0)
* Run assemblies through pipeline to identify MT sequences, adapter contamination, and viral/bacterial/fungal sequences
  * Run by Kerstin Howe w/ a pipeline developed by Kerstin and James Torrance for the Tree of Life Programme at the Wellcome Sanger Institute
* Mask adapter sequences 
  * Use union of sequences found by decontamination pipeline and by mapping the PacBio SMRTBell dimer against the assemblies w/ minimap2
* Remove Mitochondrial and contaminated contigs from assemblies
* Add the "best" MT sequence back into the assembly
  * best MT sequence defined by searching for the highest DP alignment score from minimap2 alignment against humand chrM reference
* Rename contigs to {sample_id}#{haplotype}#original_contig_id
  * {haplotype} is 1 or 2 for paternal or maternal, respectively

------------------
### Automated QC

**After assembly masking, decontamination, and MT correction, assemblies underwent automated QC in AnVIL with the following tools:**
* [asmgene](https://github.com/lh3/minimap2)
* [dipcall v0.1](https://github.com/lh3/dipcall/tree/v0.1)
* [dipcall v0.2](https://github.com/lh3/dipcall/tree/v0.2)
* [merqury](https://github.com/marbl/merqury)
* [QUAST](https://sourceforge.net/projects/quast/files/)
* [YAK](https://github.com/lh3/yak)

Select metrics were extracted and placed into the `automated_qc_results/` directory of this repo: raw values are in a CSV alongside charts for N50, hamming/switch error rates, QV values, and contig counts. Full results from automated QC are included next to the assemblies in both the AWS and GCP HPRC buckets.

**For more information about the automated QC, please see the QC workflows in the [HPP GitHub Repo](https://github.com/human-pangenomics/hpp_production_workflows).**
* [standard_qc](https://github.com/human-pangenomics/hpp_production_workflows/blob/master/QC/wdl/workflows/standard_qc.wdl)
* [standard_qc_no_qv](https://github.com/human-pangenomics/hpp_production_workflows/blob/master/QC/wdl/workflows/standard_qc_no_qv.wdl) 
  * run when no child Illumina data is available

------------------

### Data Layout
Assemblies are available in the working directory of the HPRC [S3](https://s3-us-west-2.amazonaws.com/human-pangenomics/index.html?prefix=working/) and [GCP](https://console.cloud.google.com/storage/browser/fc-4310e737-a388-4a10-8c9e-babe06aaf0cf/working?authuser=0) buckets and is organized by each trios' child sample ID:
```
 ── working/
    └── HPRC/
        └── HG00438/
            └── raw_data/ 
            └── assemblies/
                └── year1_f1_assembly_v2_genbank/
                    └── HG00438.maternal.f1_assembly_v2_genbank.fa.gz
                    └── HG00438.paternal.f1_assembly_v2_genbank.fa.gz              
                └── year1_freeze_assembly_v2/
                    └── HG00438.maternal.f1_assembly_v2.fa.gz
                    └── HG00438.paternal.f1_assembly_v2.fa.gz
                    └── assembly_qc/  
                        └── asmgene/
                        └── dipcall_v0.1/
                        └── dipcall_v0.2/
                        └── merqury/
                        └── quast/
                        └── yak/

```

Note that the current version of the assemblies is under `year1_f1_assembly_v2_genbank/`, but prior versions are also listed. Automated QC for the assemblies is under `year1_freeze_assembly_v2/` since the QC was run on that version of the assemblies.

Raw hifiasm output (including GFAs) are included for the currently assembly release in each sample's working area under `assemblies/hifiasm_v0.14_raw/`.

A complete list of the paths to the assembly fastas in S3/GCP can be found in the `assembly_index/` directory of this repo.

AnVIL users can access the data stored in GCP through the public [AnVIL_HPRC workspace](https://app.terra.bio/#workspaces/anvil-datastorage/AnVIL_HPRC). Alternatively, data can be accessed directly from AWS or GCP, and data stored in the HPRC S3 Bucket can be accessed without egress fees.

------------------

### Known Issues

* **outstanding** Screens run on the genbank version of the assemblies did not identify all contaminating contigs. A list of contigs that are likely contamination is provided in `genbank_changes/y1_genbank_remaining_potential_contamination.txt`. Note that the assembly for HG02145 (paternal) has a large number of potential EBV contigs which have not been verified.
* **fixed:** HG002 maternal contig h2tg000045l had a misjoin of chr1 and chr2 along with a large false duplication. Sample was reassembled using a new (v0.14.1) version of hifisasm to fix the misjoin issue. New assemblies released under v2.1.
* **outstanding:** HG01071 does not produce a MT contig
* **manually fixed:** HG02080 (maternal) had a misjoin in h2tg000053l
  * h2tg000053l_1 : h2tg000053l [0, 41506502]
  * h2tg000053l_2 : h2tg000053l [63683095, 92488067]
* **outstanding**: HG02080 (paternal) has a contig (h1tg000073l) where 4mb aligns to chr17 and 7mb aligns to chr19. It is unclear if this is a misjoin so it was not manually broken.
* **manually fixed:** The original paternal v2 assemblies for HG01123 & HG01358 each had a misjoin and a false duplication. This was fixed manually and released under v2.1
  * Only these two samples (and HG002) have v2.1 
  * Note that the QC was run w/ the name v2 but v2.1 data was used for these two samples 
  * Below are the break points:
    * HG01123: remove [94439457, contig end) of h1tg000013l 
    * HG01358: For h1tg000058l, keep [0, 95732608), remove [95732608, 150395342), keep [150395342, contig end)


### Change Log

```
* Jan 15, 2021: internal release of v1 assemblies
* Mar 08, 2021: internal release of v2 assemblies
  * Added two new samples
  * Added HiFiAdapterFilt preprocessing of HiFi data
  * Switched to Hifiasm v0.14
  * Added masking based on minimap2 alignments of SMRTBell adapter dimer
* Mar 18, 2021: fixed misjoin and false duplications in paternal assemblies for HG01123 & HG01358
* Apr 06, 2021: fixed misjoin and false duplication in HG002 maternal contig.
* Jun 23, 2021: internal release of v2 assemblies (Genbank version)
  * Genbank identified and dropped around 3k contigs (almost all EBV)
  * Assemblies were renamed to reflect their accesions in Genbank
  * Some assemblies were trimmed to remove leading/trailing N's from adapter hard masking
```

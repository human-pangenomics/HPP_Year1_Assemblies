## Year 1 Assemblies

![White Logo](https://s3-us-west-2.amazonaws.com/human-pangenomics/backup/logo-proof-full.png)

*This repo describes assemblies produced by the [Human Pangenome Reference Consortium](https://humanpangenome.org/) from year 1 data. Assemblies for 47 samples are available. For information about data reuse and publicating with HPRC data please see the HPRC's [Data Use Protocol](https://humanpangenome.org/data-use-protocol/).*

**Note: The assemblies contained in the HPRC [S3](https://s3-us-west-2.amazonaws.com/human-pangenomics/index.html?prefix=working/) and [GCP](https://console.cloud.google.com/storage/browser/fc-4310e737-a388-4a10-8c9e-babe06aaf0cf/working?authuser=0) buckets referred to in this repo have not been fully QC'd, are not published, and many have known issues.**

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

Raw hifiasm output (including GFAs) are included for the currently assembly release in each sample's working area under `assemblies/hifiasm_v0.14_raw/`.

A complete list of the paths to the assembly fastas in S3/GCP can be found in the `assembly_index/` directory of this repo.

AnVIL users can access the data stored in GCP through the public [AnVIL_HPRC workspace](https://app.terra.bio/#workspaces/anvil-datastorage/AnVIL_HPRC). Alternatively, data can be accessed directly from AWS or GCP, and data stored in the HPRC S3 Bucket can be accessed without egress fees.

------------------

### Known Issues

* **outstanding:** HG002 maternal contig h2tg000045l has a misjoin of chr1 and chr2 along with a large false duplication. HG002 is currently being reassembled with all PacBio HiFi v2 data and a newer version of hifiasm to fix these issues. 
* **outstanding** HG01071 does not produce a MT contig
* **manually fixed:** HG02080 had a misjoin in h2tg000053l
  * h2tg000053l_1 : h2tg000053l [0, 41506502]
  * h2tg000053l_2 : h2tg000053l [63683095, 92488067]
* **manually fixed:** The original paternal v2 assemblies for HG01123 & HG01358 each had a misjoin and a false duplication. This was fixed manually and released under v2.1
  * Only these two samples have v2.1 
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
```

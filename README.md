# CSHL_seqtec_2022 Live-coding Resource

## Step 1: Get data

The data is hosted on DropBox, you can download it with the following `wget` command:<br>
`wget https://www.dropbox.com/s/pwjj61dzojg2ajy/T2T_data.tar.gz?dl=1 -O T2T_data.tar.gz`

Unpack the data:<br>
`tar -xzf T2T_data.tar.gz`

There are two directories in `T2T_data`:
1. `alignments`: contains chr21 alignments for two samples on GRCh38 and CHM13v2
2. `references`: contains fastas, gffs, and corresponding indicies for GRCh38 and CHM13v2

### Where did the data come from?
1. `CHM13v2.chr21.fasta`: Full genome fasta downloaded from https://www.ncbi.nlm.nih.gov/assembly/GCA_009914755.4. Then, run `samtools faidx <file> chr21` to isolate chr21.
2. `GRCh38.chr21.fasta`: Full genome fasta downloaded from https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.40. Then, run `samtools faidx <file> chr21` to isolate chr21.
3. `CHM13v2.chr21.gencode.v35.gff3.gz`. Full GFF downloaded from https://github.com/marbl/CHM13#downloads (the [Sorted GFF](https://s3-us-west-2.amazonaws.com/human-pangenomics/T2T/CHM13/assemblies/annotation/chm13v2.0_GENCODEv35_CAT_Liftoff.vep.gff3.gz) file under `Assembly v2.0`. Then, run `tabix -p gff <file>` and `tabix <file> chr21` to isolate chr21.
4. `GRCh38.chr21.gencode.v35.gff3.gz`. Full GFF downloaded from https://www.gencodegenes.org/human/ under [Comprehensive gene annotation](https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_42/gencode.v42.annotation.gff3.gz). Then, run `tabix -p gff <file>` and `tabix <file> chr21` to isolate chr21.
5. The CHM13v2 cram files were downloaded from the T2T AnVil workspace [here](https://anvil.terra.bio/#workspaces/anvil-datastorage/AnVIL_T2T). Navigate to the "Data" tab at the top, and then the "Participant" data table via the sidebar on the left. The crams and cram indicies are in the `cram` and `cram_index` columns, respectively. Then, run `samtools view -ShC -T <ref.fasta> <cram> chr21` to isolate chr21. **WARNING**: The CRAM files are quite large (upwards of 10GB) and require an AnVilpayline to download.
6. The GRCh38 cram files were downloaded from SRA [here](https://ftp.sra.ebi.ac.uk/vol1/run/). You'll have to get the SRA accession for each sample, and navigate to the appropriate directory in SRA. For example, sample HG00116 has accession: `ERR3240132`; the path to the cram file is https://ftp.sra.ebi.ac.uk/vol1/run/ERR324/ERR3240132/. Then, run `samtools view -ShC -T <ref.fasta> <cram> chr21` to isolate chr21.

## Step 2: Variant calling with `freebayes`

### 2.1: Run variant calling

Start a `screen` session to run CHM13v2 variant calling:<br>
`screen -S freebayes_CHM13v2`

Within this screen session, run freebayes on the CHM13v2 alignments. We'll use `time` to keep track of how long variant calling took:<br>
`time freebayes -f T2T_data/references/CHM13v2.chr21.fasta --genotype-qualities T2T_data/alignments/*.CHM13v2.chr21.cram > CHM13v2.chr21.vcf`

Start another `screen` session to run GRCh38 variant calling:<br>
`screen -S freebayes_GRCh38`

Within this screen session, run freebayes on the GRCh38 alignments (using `time` as before):<br>
`time freebayes -f T2T_data/references/GRCh38.chr21.fasta --genotype-qualities T2T_data/alignments/*.GRCh38.chr21.cram > GRCh38.chr21.vcf`

Once `freebayes` is done running, we shouldn't need these `screen` sessions anymore.

### 2.2: Filter to high-quality variants

Filter CHM13v2 vcf to variants with 99% probability:<br>
`vcffilter -f "QUAL > 20" CHM13v2.chr21.vcf > CHM13v2.chr21.filtered.vcf`

Filter GRCh38 vcf to variants with 99% probability:<br>
`vcffilter -f "QUAL > 20" GRCh38.chr21.vcf > GRCh38.chr21.filtered.vcf`

### 2.3: Zip and index the filtered vcfs

Zip the filtered CHM13v2 vcf:<br>
`bgzip CHM13v2.chr21.filtered.vcf`<br>
`tabix -p vcf CHM13v2.chr21.filtered.vcf.gz`

Zip the filtered GRCh38 vcf:<br>
`bgzip GRCh38.chr21.filtered.vcf`<br>
`tabix -p vcf GRCh38.chr21.filtered.vcf.gz`<br>


## Step 3: Get alignment statistics with `samtools stats`

Get alignment statistics for CHM13v2 bams:<br>
`samtools stats -r T2T_data/references/CHM13v2.chr21.fasta -@ 4 T2T_data/alignments/HG00116.CHM13v2.chr21.cram > HG00116.CHM13v2.chr21.cramstats.txt`<br>
`samtools stats -r T2T_data/references/CHM13v2.chr21.fasta -@ 4 T2T_data/alignments/HG01509.CHM13v2.chr21.cram > HG01509.CHM13v2.chr21.cramstats.txt`

Get alignment statistics for GRCh38 bams:<br>
`samtools stats -r T2T_data/references/GRCh38.chr21.fasta -@ 4 T2T_data/alignments/HG00116.GRCh38.chr21.cram > HG00116.GRCh38.chr21.cramstats.txt`<br>
`samtools stats -r T2T_data/references/GRCh38.chr21.fasta -@ 4 T2T_data/alignments/HG01509.GRCh38.chr21.cram > HG01509.GRCh38.chr21.cramstats.txt`


## Step 4: Get variant calling statistics with `bcftools stats`

Get variant calling statistics for CHM13v2 vcf:<br>
`bcftools stats -s- CHM13v2.chr21.filtered.vcf.gz > CHM13v2.chr21.filtered.stats.txt`

Get variant calling statistics for GRCh38 vcf:<br>
`bcftools stats -s- GRCh38.chr21.filtered.vcf.gz > GRCh38.chr21.filtered.stats.txt`


## Step 5: Plot alignment and variant calling statistics

Plot variant counts, as well as proper pairing, and mismatch rate stats for alignment, as shown in the `analyze_stats.ipynb` notebook.

## Step 6: Plot KCNE1 alignments in IGV

First, either `scp` the crams to the local, or just `wget` them straight to the local.

Using `grep` determine the coordinates of KCNE1 from the Gencode gff files:<br>
`zgrep "KCNE1" CHM13v2.chr21.gencode.v35.gff3.gz | head -n 1 | cut -f 1,4,5`<br>
`zgrep "KCNE1" GRCh38.chr21.gencode.v35.gff3.gz | head -n 1 | cut -f 1,4,5`

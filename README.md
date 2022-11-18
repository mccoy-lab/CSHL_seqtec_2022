# CSHL_seqtec_2022 Live-coding Resource

## Step 1: Get data

The data is hosted on DropBox, you can download it with the following `wget` command:<br>
`wget https://www.dropbox.com/s/pwjj61dzojg2ajy/T2T_data.tar.gz?dl=1 -O T2T_data.tar.gz`

Unpack the data:<br>
`tar -xzf T2T_data.tar.gz`


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
`samtools stats -r T2T_data/references/CHM13v2.chr21.fasta -@ 4 T2T_data/alignments/HG00116.CHM13v2.chr21.cram > HG00116.CHM13v2.chr21.cramstats.txt`
`samtools stats -r T2T_data/references/CHM13v2.chr21.fasta -@ 4 T2T_data/alignments/HG01509.CHM13v2.chr21.cram > HG01509.CHM13v2.chr21.cramstats.txt`

Get alignment statistics for GRCh38 bams:<br>
`samtools stats -r T2T_data/references/GRCh38.chr21.fasta -@ 4 T2T_data/alignments/HG00116.GRCh38.chr21.cram > HG00116.GRCh38.chr21.cramstats.txt`
`samtools stats -r T2T_data/references/GRCh38.chr21.fasta -@ 4 T2T_data/alignments/HG01509.GRCh38.chr21.cram > HG01509.GRCh38.chr21.cramstats.txt`
# CSHL_seqtec_2022 Live-coding Resource

## Step 1: Get data

The data is hosted on DropBox, you can download it with the following `wget` command:<br>
`wget https://www.dropbox.com/s/pwjj61dzojg2ajy/T2T_data.tar.gz?dl=1 -O T2T_data.tar.gz`

Unpack the data:<br>
`tar -xzf T2T_data.tar.gz`


## Step 2: Get `freebayes` running

Start a `screen` session to run CHM13v2 variant calling:<br>
`screen -S freebayes_CHM13v2`

Within this screen session, run freebayes on the CHM13v2 alignments:<br>
`freebayes -f T2T_data/references/CHM13v2.chr21.fasta --genotype-qualities T2T_data/alignments/*.CHM13v2.chr21.cram > CHM13v2.chr21.vcf`

Start another `screen` session to run GRCh38 variant calling:<br>
`screen -S freebayes_GRCh38`

Within this screen session, run freebayes on the GRCh38 alignments:<br>
`freebayes -f T2T_data/references/GRCh38.chr21.fasta --genotype-qualities T2T_data/alignments/*.GRCh38.chr21.cram > GRCh38.chr21.vcf`


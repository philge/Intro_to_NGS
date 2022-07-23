## Intro_to_NGS

### Run Docker container
```
docker run -it -v /home/ppw5y/philge/course/:/course/ ngs_tools /bin/bash
```

### File Types  
#### FASTA
```
mkdir -p assembly/quast/
cd assembly/quast/
Rhodobacter genome sequence
wget -c https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/012/905/GCF_000012905.2_ASM1290v2/GCF_000012905.2_ASM1290v2_genomic.fna.gz
ls –lh
gunzip GCF_000012905.2_ASM1290v2_genomic.fna.gz
less GCF_000012905.2_ASM1290v2_genomic.fna
cd ..
```

#### FASTQ
```
mkdir data
cd data
Rhodobacter HiSeq data
wget -c https://ccb.jhu.edu/gage_b/datasets/R_sphaeroides_HiSeq.tar.gz
tar -xvf R_sphaeroides_HiSeq.tar.gz
less raw/insert_220_1.fastq
cd ..
```

#### SAM
```
cd quast
conda install -c bioconda bwa
bwa index GCF_000012905.2_ASM1290v2_genomic.fna
bwa mem -t 60 GCF_000012905.2_ASM1290v2_genomic.fna ../data/trimmed/insert_220_1.fastq ../data/trimmed/insert_220_2.fastq 1>GCF_000012905.2_ASM1290v2_genomic.sam 2>bwa_mem.e
conda install -c bioconda samtools
samtools view GCF_000012905.2_ASM1290v2_genomic.sam | less
```

#### BAM
```
mkdir ../temp
samtools sort -@ 60 -T ../temp/ -O BAM -o GCF_000012905.2_ASM1290v2_genomic.bam GCF_000012905.2_ASM1290v2_genomic.sam 1>samtools.o 2>samtools.e
samtools index GCF_000012905.2_ASM1290v2_genomic.bam
```

#### CRAM
```
samtools view -C -T GCF_000012905.2_ASM1290v2_genomic.fna -o GCF_000012905.2_ASM1290v2_genomi.cram GCF_000012905.2_ASM1290v2_genomic.bam
````

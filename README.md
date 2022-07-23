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

#### GFF
```
wget -c https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/012/905/GCF_000012905.2_ASM1290v2/GCF_000012905.2_ASM1290v2_genomic.gff.gz
gunzip GCF_000012905.2_ASM1290v2_genomic.gff.gz
less GCF_000012905.2_ASM1290v2_genomic.gff
cd ..
```

### Quality Control  
#### FastQC
```
cd data/raw/
conda install -c bioconda fastqc
fastqc -t 60 -q *.fastq
firefox --no-remote insert_220_1_fastqc.html &
cd ..
```

### Genome Assembly
#### Spades
```
mkdir spades/
cd spades/
conda install -c bioconda spades
spades.py -t 60 --pe1-1 ../data/trimmed/insert_220_1.fastq --pe1-2 ../data/trimmed/insert_220_2.fastq -o Spades_Rhodobacter 1>spades.o 2>spades.e
cd ../
```

#### Abyss
```
mkdir abyss
cd abyss
conda install -c bioconda abyss
export OMPI_ALLOW_RUN_AS_ROOT=1
export OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1 
abyss-pe k=31 np=30 name=asm in=' ../data/trimmed/insert_220_1.fastq ../data/trimmed/insert_220_2.fastq' 1>abyss.o 2>abyss.e
#real    12m46.002s
k: size of k-mer
np: number of MPI processes
```

#### QUAST evaluation
```
cd ../quast
#Installation as in http://quast.sourceforge.net/docs/manual.html#sec3 
quast.py ../spades/Spades_Rhodobacter/contigs.fasta ../abyss/asm-contigs.fa -t 60 --labels " spades, abyss" -r GCF_000012905.2_ASM1290v2_genomic.fna -g GCF_000012905.2_ASM1290v2_genomic.gff 1>quast.o 2>quast.e
#real    0m13.402s
```

#### Polish spades assembly
```
cd ../spades/Spades_Rhodobacter/
bwa index contigs.fasta
bwa mem -t 60 contigs.fasta ../../data/trimmed/insert_220_1.fastq ../../data/trimmed/insert_220_2.fastq 2>bwa_mem.e | samtools sort -@ 60 -T ../../temp/ -O BAM -o bwa_alignment_sorted.bam - 1>samtools.o 2>samtools.e
#real    0m44.949s
samtools index bwa_alignment_sorted.bam
conda install -c bioconda pilon
pilon --genome contigs.fasta --frags bwa_alignment_sorted.bam --outdir polished_assembly --output polished_assembly --changes 1>pilon.o 2>pilon.e
#real    3m8.137s
--changes: If specified, a file listing changes in the <output>.fasta will be generated.
```

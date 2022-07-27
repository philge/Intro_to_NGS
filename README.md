## Intro_to_NGS

### Pull Docker image
```
docker pull philge/ngs_tools
```

### Run Docker container
```
docker run -it -v C:\Users\philge1.philip\Desktop\course\:/course/ philge/ngs_tools /bin/bash
cd course
```

### File Types  
#### FASTA
```
mkdir -p assembly/quast/
cd assembly/quast/
#Rhodobacter genome sequence
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
#Rhodobacter HiSeq data
wget -c https://ccb.jhu.edu/gage_b/datasets/R_sphaeroides_HiSeq.tar.gz
#real    8m37.836s
tar -xvf R_sphaeroides_HiSeq.tar.gz
#real    2m47.059s
less raw/insert_220_1.fastq
cd ..
```

#### SAM
```
cd quast
#conda install -c bioconda bwa
bwa index GCF_000012905.2_ASM1290v2_genomic.fna
bwa mem -t 6 GCF_000012905.2_ASM1290v2_genomic.fna ../data/trimmed/insert_220_1.fastq ../data/trimmed/insert_220_2.fastq 1>GCF_000012905.2_ASM1290v2_genomic.sam 2>bwa_mem.e
#real    18m49.930s
#conda install -c bioconda samtools
samtools view GCF_000012905.2_ASM1290v2_genomic.sam | less
```

#### BAM
```
mkdir ../temp
samtools sort -@ 6 -T ../temp/ -O BAM -o GCF_000012905.2_ASM1290v2_genomic.bam GCF_000012905.2_ASM1290v2_genomic.sam 1>samtools.o 2>samtools.e
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
#conda install -c bioconda fastqc
fastqc -t 60 -q *.fastq
#real    2m0.347s
firefox --no-remote insert_220_1_fastqc.html &
cd ../../
```

### Genome Assembly
#### Spades
```
mkdir spades/
cd spades/
#conda install -c bioconda spades
spades.py -t 6 --pe1-1 ../data/trimmed/insert_220_1.fastq --pe1-2 ../data/trimmed/insert_220_2.fastq -o Spades_Rhodobacter 1>spades.o 2>spades.e
cd ../
```

#### Abyss
```
mkdir abyss
cd abyss
#conda install -c bioconda abyss
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
#conda install -c bioconda pilon
pilon --genome contigs.fasta --frags bwa_alignment_sorted.bam --outdir polished_assembly --output polished_assembly --changes 1>pilon.o 2>pilon.e
#real    3m8.137s
--changes: If specified, a file listing changes in the <output>.fasta will be generated.
```

#### Polish ABYSS assembly
```
cd ../../abyss/
bwa index asm-contigs.fa
bwa mem -t 60 asm-contigs.fa ../data/trimmed/insert_220_1.fastq ../data/trimmed/insert_220_2.fastq 2>bwa_mem.e | samtools sort -@ 60 -T ../temp/ -O BAM -o bwa_alignment_sorted.bam - 1>samtools.o 2>samtools.e
#real    0m43.599s
samtools index bwa_alignment_sorted.bam
pilon --genome asm-contigs.fa --frags bwa_alignment_sorted.bam --outdir polished_assembly --output polished_assembly --changes 1>pilon.o 2>pilon.e
#real    3m15.166s
	--changes: If specified, a file listing changes in the <output>.fasta will be generated.
```

#### QUAST
```
cd ../quast
quast.py ../spades/Spades_Rhodobacter/polished_assembly/polished_assembly.fasta ../abyss/polished_assembly/polished_assembly.fasta -t 60 --labels " spades_polish, abyss_polish" -r GCF_000012905.2_ASM1290v2_genomic.fna -g GCF_000012905.2_ASM1290v2_genomic.gff 1>quast_polish.o 2>quast_polish.e
#real    0m13.402s
cd ../../
```

### Variant calling workflow
#### Create Index
```
mkdir -p variant_calling/data/
cd variant_calling/data/
wget -c ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR741/SRR741385/SRR741385_1.fastq.gz
wget -c ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR741/SRR741385/SRR741385_2.fastq.gz
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg38/chromosomes/chr2.fa.gz
gunzip chr2.fa.gz
vi region.bed
chr2    136545000       136617000
#conda install -c bioconda bedtools
bedtools getfasta -fi chr2.fa -bed region.bed -fo chr2_region.fasta
#edit header, update as only chr2
vi chr2_region.fasta
bwa index -a bwtsw chr2_region.fasta
samtools faidx chr2_region.fasta
#conda install -c bioconda gatk4
#Generate a GATK sequence dictionary
gatk --java-options -Xmx7g CreateSequenceDictionary -R chr2_region.fasta  -O chr2_region.dict 1>gatk_dict.o 2>gatk_dict.e
```

#### Aligning reads BWA mem
```
bwa mem -R "@RG\tID:readgroup_HG00097\tPU:lanex_flowcellx\tSM:HG00097\tLB:libraryx\tPL:illumina" -t 60 chr2_region.fasta SRR741385_1.fastq.gz SRR741385_2.fastq.gz | samtools sort -@ 60 -o HG00097.bam
samtools index HG00097.bam
cd ..
wget -c https://data.broadinstitute.org/igv/projects/downloads/2.13/IGV_Linux_2.13.2_WithJava.zip
unzip IGV_Linux_2.13.2_WithJava.zip
cd IGV_Linux_2.13.2/
bash igv.sh
Qns:
What is the read length?
Approximately how many reads cover an arbitrary position in the genomic region we are looking at?
Which RefSeq Genes are located within the region chr2:136545000-136617000?
```

#### Variant Calling HaplotypeCaller
```
gatk --java-options -Xmx7g HaplotypeCaller \
-R chr2.fasta \
-I HG00097.bam \
-O HG00097.vcf
#To view the header line
grep '#CHROM' HG00097.vcf
# To view the meta-information lines describing the INFO column
grep '##INFO' HG00097.vcf
To view the meta-information lines describing the FORMAT column
grep '##FORMAT' HG00097.vcf
To look at the details of one specific genetic variant at position 2:136545844
grep '136545844' HG00097.vcf
```
Load the file HG00097.vcf into tracks window of IGV as you did with the HG00097.bam file earlier (load the bam file as well if it is not already loaded). You will now see all the variants called in HG00097. You can view variants in the LCT gene by typing the gene name in the search box, and you can look specifically at the variant at position chr2:136545844 by typing that position in the search box.

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

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
cd ..![image](https://user-images.githubusercontent.com/7710973/180585312-96030f6c-620e-4c7a-85c8-605576252d6b.png)
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
![image](https://user-images.githubusercontent.com/7710973/180585342-f300567e-7442-42d2-84c5-690f878ebfd1.png)
```

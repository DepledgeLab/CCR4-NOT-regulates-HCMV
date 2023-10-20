# CCR4-NOT-regulates-HCMV
Datasets associated with our recent study entitled ['CCR4-NOT differentially controls host vs. virus poly(A)-tail length and regulates HCMV infection'](https://www.embopress.org/doi/full/10.15252/embr.202256327) (PMID: 37846490) are available here.


### Bedgraph files were generated using trimgalore v0.5, bbmap v38.25, samtools v1.9, and bedtools v2.27.1
```
### Trim adaptors and quality
trim_galore --paired --quality 30 --length 30 -o /trimmed infile_R1.fastq.gz infile_R2.fastq.gz

### Align vs combined human and HCMV strain AD169 reference
bbmap.sh ref=HCMV-AD169_HG38.fasta intronlen=30 pairedonly=t pairlen=200000 maxindel=75000 in=/trimmed/infile_R1_val_1.fq.gz in2=/trimmed/infile_R2_val_2.fq.gz outm=aligned.sam nodisk ambiguous=random sam=1.3

### Sort and Index
samtools view -bS -o aligned.bam aligned.sam
samtools sort -o aligned.sorted.bam aligned.bam  
samtools index aligned.sorted.bam

### Retrieve only HCMV alignments
samtools view -b -o hcmv.aligned.bam AD169:1-229050

### Sample HCMV alignments (500000 read pairs)
reformat.sh in=hcmv.aligned.bam out=hcmv.aligned.sampled.bam samplereadstarget=500000 ref=HCMV-AD169.fasta overwrite=TRUE

### Sort, index, and produce bedgraph
samtools sort -o hcmv.aligned.sampled.sorted.bam hcmv.aligned.sampled.bam
samtools index hcmv.aligned.sampled.sorted.bam
samtools view -b hcmv.aligned.sampled.sorted.bam | genomeCoverageBed -ibam stdin -bg -split > hcmv.aligned.sampled.bedgraph
```


### Poly(A) tail length estimates were generated using nanopolish v0.13.3 from transcriptome level alignments against the HCMV strain TB40E-cloneBAC4  transcriptome and a human database of mRNAs and lncRNAs derived from [Gencode v37](https://www.gencodegenes.org/human/release_37.html)
```
nanopolish polya --threads=8 --reads=in.fastq --bam=hcmv.sorted.bam --genome=TB40E-cloneBAC4.fasta > hcmv.polyA.tsv
nanopolish polya --threads=8 --reads=in.fastq --bam=human.sorted.bam --genome=gencode.v37.pc.lncRNA_transcripts.fa > human.polyA.tsv
```


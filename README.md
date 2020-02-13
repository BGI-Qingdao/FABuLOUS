# TGS-GapCloser

[![DOI](https://zenodo.org/badge/183120917.svg)](https://zenodo.org/badge/latestdoi/183120917)

A gap-closing software tool that uses error-prone long reads generated by third-generation-sequence techniques (Pacbio, Oxford Nanopore, etc.) or preassembled contigs to fill N-gap in the genome assembly.  

- Both raw reads and pre-error-corrected reads are acceptable as input.

- If only raw long reads are provided, it polishes raw TGS reads by calling Racon.

- If additional NGS short reads are available, it polishes raw TGS reads by calling Pilon.

*Notice : only fasta format of TGS reads is acceptable.*

## Dependencies
1. gcc 4.4+
2. make 3.8+

## Installation

### Download 
```
git clone  https://github.com/BGI-Qingdao/TGS-GapCloser.git YOUR-INSTALL-DIR
```

### Compile

#### configure minimap2

- If you have already installed a minimap2, then link it here

```
rm -rf YOUR-INSTALL-DIR/minimap2
ln -s MINIMAP2-PATH YOUR-INSTALL-DIR/
```
- Otherwise you need to download it by:

```
cd YOUR-INSTALL-DIR
git submodule init
git submodule update
```

#### compile main src

```
cd YOUR-INSTALL-DIR/main_src_ont
make
```

## Usage 

```
Usage:
      TGS-GapCloser.sh --scaff SCAFF_FILE --reads TGS_READS_FILE --output OUT_PREFIX [options...]
      required:
          --scaff     <draft scaffolds>      input draft scaffolds.
          --reads     <TGS reads>     input TGS reads.
          --output    <output prefix>      output prefix.
     ## error correction module
          --ne                             do not execute error correction.
          or
          --racon     <racon>              installed racon. Can be installed following https://github.com/isovic/racon
          or
          --pilon     <pilon>              installed pilon jar package. Can be downloaded from https://github.com/broadinstitute/pilon/releases/download/v1.23/pilon-1.23.jar
          --java      <java>               installed java.
          --ngs       <ngs_reads>          input NGS reads used for pilon.
          --samtools  <samtools>           installed samtools.
          
          
      optional:
          --tgstype   <pb/ont>             TGS type. ont by default.
          --min_idy   <float>              minimum identity for filtering candidate sequences.
                                           0.3 for ont by default.
                                           0.2 for pb by default.
          --min_match <int>                minimum matched length for filtering candidate sequences.
                                           300 for ont by default.
                                           200 for pb by default.
          --thread    <int>                number of threads uesd. 16 by default.
          --pilon_mem <int>                memory used for pilon, passing to -Xmx.  can use “m” or “M” for MB, or “g” or “G” for GB. 300G by default.
          --chunk     <int>                split candidates into # of chunks to separately correct errors. 3 by default.
          --p_round   <int>                iteration number for pilon error-correction. 3 by default.
          --r_round   <int>                iteration number for racon error-correction. 1 by default.
```

> Notice : only fasta format TGS reads is supperted and fastq format will lead to program carshing !

## Examples

### an example of pre-corrected TGS reads without error correction 

```
YOUR-INSTALL-DIR/TGS-GapCloser.sh  \
        --scaff  scaffold-path/scaffold.fasta \
        --reads  tgs-reads-path/tgs.reads.fasta \
        --output test_ne  \
        --ne \
        >pipe.log 2>pipe.err
```

### an example of raw ONT reads with error correction using long reads only

```
YOUR-INSTALL-DIR/TGS-GapCloser.sh  \
        --scaff  scaffold-path/scaffold.fasta \
        --reads  tgs-reads-path/tgs.reads.fasta \
        --output test_racon \
        --racon  racon-path/bin/racon \
        >pipe.log 2>pipe.err
```

### an example of raw ONT reads with error correction using NGS reads

```
YOUR-INSTALL-DIR/TGS-GapCloser.sh  \
        --scaff  scaffold-path/scaffold.fasta \
        --reads  tgs-reads-path/tgs.reads.fasta \
        --output test_pilon \
        --pilon  pilon-path/pilon-1.23.jar  \
        --ngs    ngs-reads-path/ngs.reads.fastq.gz  \
        --samtools samtools-path/bin/samtools  \
        --java    java-path/bin/java \
        >pipe.log 2>pipe.err
```

### Using Pacbio reads

* default TGS type is ONT, use ```--tgstype```  to change it .

```
    --tgstype ont
    to 
    --tgstype pb
```

* an example of raw Pacbio reads with error correction using long reads only

```
YOUR-INSTALL-DIR/TGS-GapCloser.sh  \
        --scaff  scaffold-path/scaffold.fasta \
        --reads  tgs-reads-path/tgs.reads.fasta \
        --output test_racon \
        --racon  raconn-path/bin/racon \
        --tgstype pb \
        >pipe.log 2>pipe.err
```

## Output

- your-prefix.scaff_seq 
    - this is the final assembly after gap filling
- your-prefix.gap_fill_details
    - details about how the final assembly was assemblied 

### format of your-prefix.gap_fill_details

#### an example:

```
>scaffold_1
1	1000	S	1000	2000
1001	1010	N
1011	1100	S	2201	2290
1101	1110	F
1111	1200	S	2301	2390
>scaffold_2
......

```
#### detailed information

1. each scaffold name is followed by its data lines.
2. a data line consists of 3 or 5 columns and describes the source of each segment in the final sequence:
    - column 1 is the segment's first bp position in the final sequence.
    - column 2 is the segment's last bp position in the final sequence.
    - column 3 is the segment's type , 'S' , 'N' or 'F'.
        - 'S' means this segment is a segment of the input sequence and this line includes other two more columns:
            - column 4 is the segment's first bp position in the input sequence.
            - column 5 is the segment's last bp position in the input sequence.
        - 'N' means this segment is a N area.
        - 'F' means this segment is a filled sequence from TGS reads.
        
        
## Citing TGS-GapCloser
If you use TGS-GapCloser in your work, please cite:
TGS-GapCloser: fast and accurately passing through the Bermuda in large genome using error-prone third-generation long reads
Mengyang Xu, Lidong Guo, Shengqiang Gu, Ou Wang, Rui Zhang, Guangyi Fan, Xun Xu, Li Deng, Xin Liu
bioRxiv 831248; doi: https://doi.org/10.1101/831248

## Contact
Any questions, please feel free to ask xumengyang@genomics.cn or guolidong@genomics.cn

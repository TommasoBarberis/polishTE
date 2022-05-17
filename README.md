# polishTE

## Introduction

**PolishTE** is a simple bash pipeline for extension and edge polishing of transposable elements consensi. The extension of TE ends step can be important, even essential, for genotyping and identifying new insertions on new genomes.

![](assets/workflow.svg)

The program take as input a putative TE sequence and the reference genome used for the annotation and it returns the polished sequence. 

More in details, the program starts with a `blastn` against the genome, if several hits come from the same fragment on the genome the sequence is marked as simple repeat, otherwise a random sampling of hits is done (by default it samples 100 hits where 25 are the longer ones). <br>
Then, sequences corresponding to hits are extracted from the genome with a flanking region (by default 1000bp for each end) using `bedtool slop`. These sequences are aligned with `mafft` and then the MSA is cleaned using `CIAlign` and a new consensus is called. <br>
If this one is close to the original sequence it means that sequence can't extend and the new sequence is returned. In contrast, if it is sufficiently different, it means that the consensus can still expand or shorten. So, a check for the ends coverage is done, if they are high covered it means that sequence can still extend and more flanking bases are added. Otherwise, if they are low covered (or not covered at all) it means that the sequences has been far too extended and the sequence need to be trimmed. <br>
After that, the extended/trimmed sequence is used in a new cycle `blastn`-`mafft`-`CIAlign` and the new consensus is compared with that of the previous iteration. The program loop until the consensus stop to be update or a time limit is reached.


## Dependencies

- `mafft` (tested version: v7.490 (2021/Oct/30))
- `blastn` (tested version: 2.9.0+)
- `samtools` (tested version: 1.10)
- `awk` (tested version: 5.0.1)
- `bedtools` (tested version: v2.27.1)
- `CIAlign` (tested version: 1.0.15)
- `python3` (tested version: 3.10.2)
    - `python-Levenshtein` (tested version: 0.12.2)
- `R` (tested version: 3.6.3)


## Installation

### Source code installation

```
git clone https://github.com/TommasoBarberis/polishTE.git
chmod +x polishTE
```

### Singularity

```
singularity pull library://tommasobarberis98/tealb/polishte
```

### Usage

#### Basic usage

```
polishTE -i seq.fasta -g ref.fasta [OPTIONS]
```

#### With singularity

```
singularity exec polishTE.sif polishTE -i seq.fasta -g ref.fasta
```

### Options

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `**-i, --input**` | FASTA file with the sequence to polish | None |
| `**-g, --genome**` | FASTA file with the reference | None |
|` -h, --help` | show the help message | None |
| `-t, --threads` | number of threads to use in the multiple alignement with `MAFFT` | 1 |
| `-o, --output` | output directory | ./ |
| `-min, --min_length` | Minmum length for `blastn` hits that will be recovered in the genome. If the option is set to 0, the program will use the half of the length of the TE as minimum length, else it will use the provided value | 0 |
| `-f, --flank`| Number of bases to extract at the flanking regions (5' and 3') of the hits | 1000pb |
| `-e, --evalue` | e-value threshold for the `blastn`| 1e-20 |
| `-l, --limit` | Maximum number of hits to take in account for the alignment. If the number of hits is less then this limit, the subsample is skipped. | 100 |
| `-k, --keep_longest` | Keep the \<k> % longest sequences among the `blastn` hits | 0.25 |
| `-ins, --max_ins_size` | Remove an insertion from the MSA if its size is less than \<ins> (range:[200-1000]) | 200 |
| `-m, --mode` | Speed mode. fast and less accurate (max 10 iterations) or slow and more accurate (max 100 iterations). When the maximum number of iteration is reached, the program is interrupted and it returns any sequence | fast |
| `-c, --min_cov` | Minimum coverage on boundaries to perform the extension | 3 |



## References

- Goubert, C., Craig, R. J., Bilat, A. F., Peona, V., Vogan, A. A., & Protasio, A. V. (2022). A beginner’s guide to manual curation of transposable elements. Mobile DNA, 13(1), 7. doi:10.1186/s13100-021-00259-7
- Katoh, K., Misawa, K., Kuma, K., & Miyata, T. (2002). MAFFT: a novel method for rapid multiple sequence alignment based on fast Fourier transform. Nucleic Acids Research, 30(14), 3059–3066. doi:10.1093/nar/gkf436
- Camacho, C., Coulouris, G., Avagyan, V., Ma, N., Papadopoulos, J., Bealer, K., & Madden, T. L. (2009). BLAST+: architecture and applications. BMC Bioinformatics, 10(1), 421. doi:10.1186/1471-2105-10-421
- Danecek, P., Bonfield, J. K., Liddle, J., Marshall, J., Ohan, V., Pollard, M. O., … Li, H. (2021). Twelve years of SAMtools and BCFtools. GigaScience, 10(2), giab008. doi:10.1093/gigascience/giab008
- Quinlan, A. R., & Hall, I. M. (2010). BEDTools: a flexible suite of utilities for comparing genomic features. Bioinformatics, 26(6), 841–842. doi:10.1093/bioinformatics/btq033
- Tumescheit, C., Firth, A. E., & Brown, K. (2022). CIAlign: A highly customisable command line tool to clean, interpret and visualise multiple sequence alignments. PeerJ, 10, e12983. doi:10.7717/peerj.12983
- Van Rossum, G., & Drake, F. L. (2009). Python 3 Reference Manual. Scotts Valley, CA: CreateSpace
- Haapala, A. (χ.χ.). python-Levenshtein: Python extension for computing string edit distances and similarities. Ανακτήθηκε από http://github.com/ztane/python-Levenshtein
- R Core Team. (2016). R: A Language and Environment for Statistical Computing. Vienna, Austria. Retrieved from https://www.R-project.org/
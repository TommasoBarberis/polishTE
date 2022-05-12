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
    - `Levenshtein` (tested version: 0.12.2)
- `R` (tested version: 3.6.3)


## Installation

### Source code installation

```
git clone https://github.com/TommasoBarberis/polishTE.git
chmod +x polishTE
```

### Singularity


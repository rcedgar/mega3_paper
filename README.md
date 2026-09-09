### Mega3 paper code and data

This documents code and data for the Mega3 paper.

### Reseek v3 accuracy benchmarking against DALI, TM-align and Reseek

[https://github.com/rcedgar/reseek_bench3](https://github.com/rcedgar/reseek_bench3)

### Muscle-3D Balibase benchmark

[https://github.com/rcedgar/reseek_bench3/tree/main/muscle3d_balibase](https://github.com/rcedgar/reseek_bench3/tree/main/muscle3d_balibase)

### StruMM analysis of RdRp and CD-NTases

[https://github.com/rcedgar/reseek_bench3/tree/main/strumm_analysis](https://github.com/rcedgar/reseek_bench3/tree/main/strumm_analysis)

Note: extract data.tar.gz to make data/ sub-directory.

### Reseek v3 source code

[https://github.com/rcedgar/reseek](https://github.com/rcedgar/reseek)

Commit `c6eb26d92132136ac54f3d06ec295f8470801a0e`

### Muscle-3D and StruMMer source code

[https://github.com/rcedgar/muscle](https://github.com/rcedgar/muscle)

Commit `2ef686a5dab3a9b02ada6f2a4f6166910f8cb9fb`

Note: Structure-based Muscle-3D and StruMMer are currently implemented in a single `muscle` binary together with amino acid alignment methods.

### Muscle-3D usage, up to a few hundred structures

<pre>
Create multiple structure alignment as aligned FASTA of aas:
    muscle -align STRUCTS -output structs.afa

Create database in .bcb format:
    reseek -convert STRUCTS -bcb structs.bcb

STRUCTS specifies structures, one of:
    NAME.bcb     # Reseek database in .bcb format (recommended)
    NAME.bca     # Reseek database in .bca format
    NAME.cal     # Reseek database in .cal format
    NAME.files   # text file with one pathname per line
    DIRNAME/     # search directory recursively for
                    # *.pdb, *.pdb.gz, *.cif, *.cif.gz, *.mmcif,
                    # *.mmcif.gz, *.files, *.cal, *.bca, *.bcb   
</pre>

### Muscle-3D usage, large datasets many hundreds or thousands

<pre>
Create multiple structure alignment as aligned FASTA of aas:
    muscle -super7 STRUCTS -distmx structs.distmx -output structs.afa

Create distance matrix:
    reseek -distmx STRUCTS -output structs.distmx
</pre>

### Create StruMM

<pre>
muscle
    -strumm_build structs.afa
    -input STRUCTS
    -output structs.strumm
    [-nocalibrate]
    [-local | -global | -semiglobal]  default -local
    [-shatter]
    [-label NAME]
    [-decoy decoy.strumm]
    [-seedmsaout structs.seed.afa]
    [-jalviewfeatures jalview_features.txt]

Required inputs:
    structs.afa  multiple structure alignment as aligned FASTA of aas
    STRUCTS      structures (as above for Muscle-3D)

Calibration:
    -nocalibrate           do not calibrate
    -shatter               estimate FP distribution by shatter-sharding
    -decoy decoy.strumm    estimate FP distribution from decoy StruMM

Alignment mode (affects calibration only):
    -local | -global | -semiglobal

Seed alignment:
     The seed alignment has match columns plus representative insert columns,
     this aa MSA is usually much shorter than the input alignment, note
     some letters are delete from insert columns so sequences may be incomplete.

NAME is saved in the StruMM file, default is base of seed alignment filename (path and extension stripped).
</pre>

### Search and align to StruMM

<pre>
muscle
    -strumm_search STRUCTS
    -strumm family.strumm
    -output hits.tsv
    -a3m structs.a3m
    [-pvalue P]

Default P-value cutoff is determined by calibration and stored in the .strumm file, -pvalue P overrides, default 0.001 if not calibrated and -pvalue not specified.

Hits file is TSV format with 1=structure label, 2=StruMM label, 3=alignment score, 4=P-value (P-value only if StruMM is calibrated).

</pre>

### A3M alignment format

A3M is a variant of the well-known A2M format introduced by SAM and later adopted by HMMer.

With A2M format, an alignment of a sequence to an HMM is representated as follows:

- Upper case letter => aligned to match state
- Lower-case letter => insert between match states
- Dash gap `-` deletion relative to the HMM (no letter for this match state)
- Dot gap `.` gap in MSA aligned to match state

The number of upper case letters plus dash gaps is exactly the input sequence length.

A3M allows dots `.` for inserts to be omitted, enabling more compact files. Dot gaps are not necessary because every sequence must have the same number of upper-case letters plus dash `-` gaps, one for each match state in the HMM. The distinction between A3M and A2M is only relevant when aligning an MSA to a model, in which case a gap in the MSA may align to an HMM match state. When aligning an individual structure to a StruMM, as with `strumm_search`, this cannot happen, there are no dot gaps and the two formats are identical. Future extensions to StruMMer will allow MSA-to-StruMM alignments.
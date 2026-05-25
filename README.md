# TEgenomeSimulator <img src="TEgenomeSimulator_logo.png" align="right" alt="" width="120" />
A tool to simulate TE mutation and insertion into a random-synthesised or user-provided genome.

This tool is released under the GPL license to promote advancements in TE research. TEgenomeSimulator was developed by [The New Zealand Institute for Plant and Food Research Ltd](https://www.plantandfood.com/en-nz/). More details about contributors, leadership and funding can be found in [Credits, Funding and Acknowledgement](https://github.com/Plant-Food-Research-Open/TEgenomeSimulator?tab=readme-ov-file#credits-funding-and-acknowledgement).

You are welcome to raise issues for inquiries regarding TEgenomeSimulator or contribute to its development and improvement. For more details, please refer to [Contributing Guideline](./.github/contributing_guide.md).

## What's new in the latest TEgenomeSimulator (v1.0.0):
- A new flag **`--to_mask`** under Custom Genome mode (`--mode 1` or `-M 1`) to enable the use of RepeatMasker for masking TEs in user-provided real genome, followed by TE removal to generate the TE-depleted genome. 
  - This option has to be used in conjunction with `--repeat2` which specifies the TE library file for repeat masking. 
  - Note that users still need to specify the TE library used for simulation via the argument `--repeat`.
  - Users may specify different TE libraries or use the same library for `--repeat` and `--repeat2`. 

- A **new distribution for the probility of sequence integrity** has been implemented for Random Synthesized Genome mode(`--mode 0` or `-M 0`) and Custom Genome mode (`--mode 1` or `-M 1`).
  - Here **sequence integrity** referes to the length ratio of inserted TE sequences relative to the consensus sequence. For example, if a TE sequence to be inserted into the synthetic genome is 800bp and the corresponding consensus sequence is 1kb, the integrity is 0.8.
  - TEgenomeSimulator now uses beta distribution to model the sequence integrity of each TE family.
  - The alpha and beta values are 0.5 and 0.7 by default to create an asymatric U-shape distribution, in which the probability of low sequence integrity (e.g. <10%) is higher than that of high integrity (e.g. >90%). You can use this [online tool](https://homepage.stat.uiowa.edu/~mbognar/applets/beta.html) to see how the distribution would look like with different alpha and beta values.
  - Users can use the parameter `-a` or `--alpha` to specify alpha value, and use `-b` or `--beta` to specify beta value.
  - The simulator also allow users to specify a fixed proportion of total TE copies to be intact TE insertion, using the parameter `-i` or `--intact`. Intact TE loci are kept in the same length as the original TE family sequence.

- A new mode, **TE Composition Approximation mode** (`--mode 2` or `-M 2`), to allow users to create a simulated genome in which the TE composition is simulated in a way to approximate the real TE make-up in the original genome. 
  - TEgenomeSimulator does this by masking and removing the original TEs using RepeatMasker with the TE library supplied via `--repeat2`, and then simulates TE mutations based on the prior information extracted from RepeatMasker output file. 
  - The rest of the steps, random and nested TE insertions are the same as the first two modes. 
  - Note that the simulator uses the TE library specified by `--repeat` argument for TE mutagenesis and insertions. Therefore it is crucial to specify the same TE library with `--repeat` and `--repeat2` for the purpose of this genome approximation simulation.

## Introduction
TEgenomeSimulator was initially built on [Matias Rodriguez & Wojciech Makałowski. Software evaluation for de novo detection of transposons. 2022 Mobil DNA](https://mobilednajournal.biomedcentral.com/articles/10.1186/s13100-022-00266-2). TEgenomeSimulator adopted and modified scripts from the original [denovoTE-eval](https://github.com/IOB-Muenster/denovoTE-eval), and further developed with several new functionalities. The following table shows the major features that were kept, modified, or created in TEgenomeSimulator.

| Features | denovoTE-eval | TEgenomeSimulator |
|:---------|:-------------:|:-----------------:|
| Random synthesized genome |  O  |  O  |
| Custom genome |  X  |  O  |
| Simulation with multiple chromosomes |  X  |  O  | 
| Automatic generation of TE mutation parameter table<sup>a</sup> |  X  |  O  |
| Automatic generation of configuration yml file<sup>a</sup> |  X  |  O  |
| TE composition approximation to real genome | X | O |
| TE mutation: copy number<sup>b</sup> |  O  |  O (simplified)  |
| TE mutation: substitution rate |  O  |  O  |
| TE mutation: INDEL rate |  O  |  O  |
| TE mutation: standard deviation of SNP |  O  |  O  |
| TE mutation: 5' fragmentation |  O (even distribution) |  O (beta distribution) |
| TE mutation: target site duplication (TSD)<sup>c</sup> |  O  |  O (enhanced)  |
| TE mutation: strandedness |  O  |  O  |
| TE gff file: TE coordinates |  O  |  O  |
| TE gff file: TE family  |  O  |  O  |
| TE gff file: TE superfamily if provided in TE lib fasta file<sup>d</sup> |  X  |  O  |
| TE gff file: TE subclass if provided in TE lib fasta file<sup>d</sup> |  X  |  O  |
| TE gff file: nucleotide identity info |  O  |  O  |
| TE gff file: fragmentation info |  O  |  O  |
| TE gff file: coordinates of disruption if TE cut by nested TE |  O  |  O  |
| TE gff file: tag to show if TE nested in other TE  |  O  |  O  |
| TE gff file: ID of associated nested/disrupted TEs  |  X  |  O  |

<sup>a</sup> denovoTE-eval requires user to provide a configuration file and a TE mutation parameter table with information such as copy number, nucleotide identity, fragmentation, TSD etc., whereas TE genomeSimulator automatically helps user to create these files.

<sup>b</sup> denovoTE-eval requires user to specify the copy number of each TE family in the TE mutation parameter table, whereas TEgenomeSimulator provides more flexibility and allows users to specify a range of integer values where the copy number of each TE family would be randomly sampled. 

<sup>c</sup> denovoTE-eval allows user to specify whether to simulate TSD using y|n with the length of TSD randomly picked between the range of 5 and 20, whereas TEgenomeSimulator tries to recognise the TE family information from the sequence headers in TE library fasta file and then decides on the TSD length based on literature. For example, TEgenomeSimulator would set "5" as the TSD length of a Copia LTR-retrotransposon but "8-9" for Mutator DNA transposon. You can check the code of [prep_sim_TE_lib.py](TEgenomeSimulaot/utils/prep_sim_TE_lib.py) for more details about TSD length setting.

<sup>d</sup> To allow TEgenomeSimulator to include TE superfamily and class/subclass information in the final TE gff file, please follow the header format `>{TE_family}#{TE_subclass}/{TE_superfamily}`, such as `>ATCOPIA10#LTR/Copia`.

### TEgenomeSimulator flowchart
<img src="TEgenomeSimulator_v1.0.0.png" width=720>

### Run modes
TEgenomeSimulator has three modes to be specified by users using the argument `-M` or `--mode`:

1) `-M 0` or`--mode 0`: **Random Synthesized Genome mode**. This mode synthesizes a random genome with multiple chromosomes that are later inserted with TEs randomly. With this mode, user must use `-c` or `--chridx` to provide a chromosome index csv file in the formate of the file [random_genome_chr_index.csv](./test/input/random_genome_chr_index.csv), which contains 3 columns separated by commas, representing the desired chromosome id, chromosome length, and GC content. The simulator uses these information to synthesize chromosome sequences, followed with the simulation of TE sequence mutaion, random TE insertion and nested TE insertion (see the blue route in the flowchart).

2) `-M 1` or`--mode 1`: **Custom Genome mode**. 
  
    a) Running this mode without `--to_mask`:
    
    It utilises a user-provided genome containing multiple chromosomes where TE bases had been removed, providing a customised genome canvas for random TE insertion (see the hollow grey route in the flowchart). When this mode is enabled, the user must use `-g` or `--genome` to provide a genome fasta file, e.g. [Dondhond.chromosomes.only.fa.nonTE.2chrs](./test/input/Donghong.chromosomes.only.fa.nonTE.2chrs.gz), which contains TE-depleted chromosome 1 and 2 from the published [_Actinidia chinensis_ var. 'Donghong'](https://doi.org/10.1016/j.molp.2022.12.022). In this fasta file, the TE sequences on the original chromosome 1 and 2 have been detected and removed. 

    b) Running this mode with `--to_mask`:

    When the flag `--to_mask` is specified, TEgenomeSimulator allows user to provide a real genome where TE nucleoties have not been masked as 'Xs'. It does this job for users by using RepeatMasker followed by TE removal. The downstream TE mutation simulation, and random and nested TE insertions are the same as in the first mode (see the solid grey route in the flowchart).

3) `-M 2` or`--mode 2`: **TE Composition Approximation mode**. It uses RepeatMaker to mask TEs and then removes TE nucleotides from the genome provided by user. Meanwhile it analyzes the TE composition in the original genome and implements this prior information to the simulation of TE mutations (see the green route in the flowchart). When this mode is enabled, the user must use `-g` or `--genome` to provide a genome fasta file, e.g. [GCF_000001735.4_TAIR10.1_genomic.fna](./test/input/GCF_000001735.4_TAIR10.1_genomic.fna.gz), which was downloaded from NCBI. Users need to specify the TE library by both `--repeat` and `--repeat2` to ensure that the simulator uses the same TE library for TE masking/removal and simulation.


### Other required input files/information
- `-p` or `--prefix`: a prefix for your simulation
- `-r` or `--repeat`: a fasta file containing TE family sequences (i.e. a TE library) to be mutated and then inserted into genome. The TE lib in our test [combined_curated_TE_lib_ATOSZM_selected.fasta](./test/input/combined_curated_TE_lib_ATOSZM_selected.fasta.gz) was created by concatenating TE library from a) _Arabidopsis thaliana_, b) _Oryza sativa_, and c) _Zea maize_ before using bbmap2 to remove exact duplicates and using CDHIT to generate a non-redundat TE lib for simulation test. The _A. thaliana_ and _Z. maize_ TE libraries were obtained from [EDTA repo (Ou et al. 2019)](https://github.com/oushujun/EDTA/tree/master/database) and the _O. sativa_ TE library was from [RepeatModeller2 (Flynn et al. 2020)](https://github.com/jmf422/TE_annotation/tree/master/benchmark_libraries/RM2).
- `r2` or `--repeat2`: the TE lib fasta file for TE masking and removal. It is required when `--to_mask` is enabled under `--mode 1` or when `--mode 2` is chosen.
- `-m` or `--maxcp`: an interger representing the maximum copy to be simulated for a TE family (only feasible in mode 0 and 1)
- `-n` or `--mincp`: an integer representing the minimum copy to be simulated for a TE family (only feasible in mode 0 and 1)
- `-c` or `--chridx`: the chromosome index file if **Random Synthesized Genome mode** (`--mode 0`) is enabled. See the previous section **Run modes**.
- `-g` or `--genome`: the genome fasta file if **Custom Genome mode** (`--mode 1`) is enabled. See the previous section **Run modes**.
- `-f` or `--frag_mode`: mode for TE fragmentation (0, 1, or 2). If frag_mode is 2, uses an integrity list read from `--te_table`, otherwise beta distribution.
- `-a` or `--alpha`: a float number specifying the alpha value for generating beta distribution. Default is 0.7 (only feasible in mode 0 and 1).
- `-b` or `--beta`: a float number specifying the beta value for generating beta distribution. Default is 0.5 (only feasible in mode 0 and 1).
- `-i` or `--insert`: a float number between 0 and 1 representing the proportion of TE copies to be intact insertion. Default is 0.001 (i.e. 0.1%) (only feasible in mode 0 and 1).
- `-s` or `--seed`: an integer to specify the seed value. Default is 1.
- `-t` or `--threads`: an integer as the number of CPUs to be used for RepeatMasker.
- `-o` or `--outdir`: the path to the output directory.
- `--te_table`: Path to an existing TE mutagenesis settings table (modes 0 and 1 only).
- `--te_scan_only`: Mode 2 only: stop after TE composition profiling and TE table generation.


### Key output files from TEgenomeSimulator
- **TElib_sim_list.table**: A tab-delimited table storing the parameters for simulating TE mutation. See example at ./test/output/ 
- **TEgenomeSimulator_{$prefix}.yml**: A configuration file for simulation. See example at ./test/output/
- **{$prefix}_genome_sequence_out.fasta**: The simulated genome fasta file with non-overlap random TE insertion.
- **{$prefix}_"_repeat_sequence_out.fasta**: The simulated TE sequences that had been inserted into the genome.
- **{$prefix}_repeat_annotation_out.gff**: The coordinates of inserted TE after non-overlap random TE insertion.
- **{$prefix}_genome_sequence_out_final.fasta**: The final simulated genome fasta file with non-overlap and nested random TE insertion.
- **{$prefix}_repeat_sequence_out_final.fasta**: The final simulated TE sequences, including the nested TEs that had been inserted into the genome.
- **{$prefix}_repeat_annotation_out_final.gff**: The final coordinates of all inserted TE adjusted after nested TE insertion.
- **TEgenomeSimulator.log**: The log file of the simulation.


## Installation
### Method 1: Using Apptainer
The Apptainer image `TEgenomeSimulator_v1.0.0.sif` includes all dependencies required for **TEgenomeSimulator v1.0.0**, including RepeatMasker (v4.1.8) and samtools. 

**Steps:**

1. **Prerequisite:** make sure you have apptainer or singularity installed in your mechine.
2. **Download the image** from [Release](https://github.com/Plant-Food-Research-Open/TEgenomeSimulator/releases/tag/v1.0.0)
3. **Test the installation** by running `apptainer run TEgenomeSimulator_v1.0.0.sif --help`

### Method 2: pip install (Python-only)
This method installs the Python packages required for **TEgenomeSimulator v0.1.0**.

>⚠️ To use `--to_mask` in **Custom Genome mode** or **TE Composition Approximation mode**, RepeatMasker and samtools must be installed separately. For full functionality, consider using Method 1.

**Steps:**

1. **Prerequisite:** make sure you've installed RepeatMasker and samtools.
2. **Clone this repo**
```bash
cd $MYDIR
git clone git@github.com:Plant-Food-Research-Open/TEgenomeSimulator.git
```
3. **Install** the python package:
```bash
cd TEgenomeSimulator
pip install .
```
4. **Test the installation** by running `tegnomesimulator --help`

### 3. Usage
The following is the arguments of **TEgenomeSimulator v1.0.0**.

```text
usage: tegenomesimulator [-h] -M {0,1,2} [-k] [-S SIF_PATH] -p PREFIX -r REPEAT [-r2 REPEAT2] [-m MAXCP] [-n MINCP] [--maxidn MAXIDN] [--minidn MINIDN] [--maxsd MAXSD] [--minsd MINSD] [-c CHRIDX]
                         [-g GENOME] [-a ALPHA] [-b BETA] [-i INTACT] [-s SEED] [-t THREADS] -o OUTDIR

main arguments of TEgenomeSimulator to simulate TE mutation and insertion into genome.

optional arguments:
  -h, --help            show this help message and exit
  -M {0,1,2}, --mode {0,1,2}
                        Mode for genome simulation (choose from 0, 1 or 2).
  -k, --to_mask         Mask and remove TE from provided genome when enabled
  -S SIF_PATH, --sif_path SIF_PATH
                        Path to dfam-tetools.sif for running RepeatMasker (required when using --to_mask).
  -p PREFIX, --prefix PREFIX
                        Prefix for output files.
  -r REPEAT, --repeat REPEAT
                        TE family fasta file for random insertion simulation.
  -r2 REPEAT2, --repeat2 REPEAT2
                        TE family fasta file for masking genome (required when using --to_mask).
  -m MAXCP, --maxcp MAXCP
                        Maximum copies of TE family (default is 10).
  -n MINCP, --mincp MINCP
                        Minimum copies of TE family (default is 1).
  --maxidn MAXIDN       The upper bound of mean sequence identity to be sampled for each TE family (default is 95; i.e. 95 percent).
  --minidn MINIDN       The lower bound of mean sequence identity to be sampled for each TE family (default is 80; i.e. 80 percent).
  --maxsd MAXSD         The upper bound of standard deviation of mean identity to be sampled for each TE family (default is 20).
  --minsd MINSD         The lower bound of standard deviation of mean identity to be sampled for each TE family (default is 1).
  -c CHRIDX, --chridx CHRIDX
                        Chromosome index file if mode 0 is selected.
  -g GENOME, --genome GENOME
                        Genome fasta file if mode 1 or 2 is selected.
  -a ALPHA, --alpha ALPHA
                        Alpha value for the beta distribution used for fragmentation simulation (default is 0.7).
  -b BETA, --beta BETA  Beta value for the beta distribution used for fragmentation simulation (default is 0.5).
  -i INTACT, --intact INTACT
                        Maximum probability of inserting intact TEs per family (default is 0.001; i.e. 0.1 percent).
  -s SEED, --seed SEED  Random seed (default is 1).
  -t THREADS, --threads THREADS
                        Threads for running RepeatMasker (default is 1)
  -o OUTDIR, --outdir OUTDIR
                        Output directory.
```

## Run TEgenomeSimulator using test data
This section provides some examples of running TEgenomeSimulator using the Apptainer image.

Before running tests, remember to unpack the compressed files in the test/input folder.
```
git clone git@github.com:Plant-Food-Research-Open/TEgenomeSimulator.git
cd TEgenomeSimulator
gzip -d -k ../test/input/*
```

### Random Synthesized Genome mode 
```bash
cd TEgenomeSimulator
prefix=test_random
chridx="../test/input/random_genome_chr_index.csv" 
repeat="../test/input/combined_curated_TE_lib_ATOSZM_selected.fasta"
max=5
min=1
outdir="../test/output"
mkdir -p $outdir

apptainer run TEgenomeSimulator_v1.0.0.sif -M 0 -p $prefix -c $chridx -r $repeat -m $max -n $min -o $outdir
```

### Custom Genome mode
#### Running without `--to_mask`
```bash
cd TEgenomeSimulator
prefix=test_custom
genome="../test/input/Donghong.chromosomes.only.fa.nonTE.2chrs"
repeat="../test/input/combined_curated_TE_lib_ATOSZM_selected.fasta"
max=5
min=1
outdir="../test/output"
mkdir -p $outdir

apptainer run TEgenomeSimulator_v1.0.0.sif -M 1 -p $prefix -g $genome -r $repeat -m $max -n $min -o $outdir
```

#### Running with `--to_mask`

- Use test files provided in the test folder: 
  - Arabidopsis genome downloaded from ncbi at https://api.ncbi.nlm.nih.gov/datasets/v2alpha/genome/accession/GCF_000001735.4/
  - Arabidopsis TE lib acquired from https://github.com/oushujun/EDTA/blob/master/database/

```bash
prefix=test_custom_tomask
genome="../test/input/GCF_000001735.4_TAIR10.1_genomic.fna"
repeat="../test/input/athrep.updated.nonredun.fasta"
outdir="../test/output"
mkdir -p $outdir

THREADS=20 # for running RepeatMasker

cd $outdir
apptainer run TEgenomeSimulator_v1.0.0.sif -M 1 --to_mask -p $prefix -g $genome -r $repeat -r2 $repeat -o $outdir -t $THREADS
```


### TE Composition Approximation mode
```bash
prefix=test_genome_approx
genome="../test/input/GCF_000001735.4_TAIR10.1_genomic.fna"
repeat="../test/input/athrep.updated.nonredun.fasta"
outdir="../test/output"
mkdir -p $outdir

THREADS=20

cd $outdir
apptainer run TEgenomeSimulator_v1.0.0.sif -M 2 -S $tetools -p $prefix -g $genome -r $repeat -r2 $repeat -o $outdir -t $THREADS
```


## References:
- [Matias Rodriguez & Wojciech Makałowski. Software evaluation for de novo detection of transposons. 2022 Mobil DNA](https://mobilednajournal.biomedcentral.com/articles/10.1186/s13100-022-00266-2)
- [denovoTE-eval: https://github.com/IOB-Muenster/denovoTE-eval](https://github.com/IOB-Muenster/denovoTE-eval)
- [Ou et al. Benchmarking transposable element annotation methods for creation of a streamlined, comprehensive pipeline. 2019 Genome Biology](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1905-y)
- [Flynn et al. RepeatModeler2 for automated genomic discovery of transposable element families. 2020 PNAS](https://doi.org/10.1073/pnas.1921046117)
- Smit, AFA, Hubley, R & Green, P. **RepeatMasker Open-4.0.** 2013-2015 <http://www.repeatmasker.org>.
- [Li et al. The Sequence Alignment/Map format and SAMtools. 2009 Bioinformatics](https://doi.org/10.1093/bioinformatics/btp352)


## Credits, Funding and Acknowledgement
- **Original Development**: TEgenomeSimulator was designed and developed by Ting-Hsuan Chen [@ting-hsuan-chen](https://github.com/ting-hsuan-chen).
- **Funding and Leadership**: This work is part of the Kiwifruit Royalty Investment Programme (KRIP)-funded _Genome Landscape_ objective, led by Susan Thomson [@cflsjt](https://github.com/cflsjt).
- **Reviewing and Testing**: Special thanks to Olivia Angeline-Bonnet [@oliviaAB](https://github.com/oliviaAB), James Bristow [@JBris](https://github.com/JBris), and Christopher Benson [@cwb14](https://github.com/cwb14) for reviewing and testing TEgenomeSimulator, and to Cecilia Deng [@CeciliaDeng](https://github.com/CeciliaDeng) and Shujun Ou [@oushujun](https://github.com/oushujun) for additional review efforts.
- **Team Contributions**: We appreciate all the advice and comments from The New Zealand Institute for Plant and Food Research Limited Bioinformatics Analysis and Bioinformatics Engineering Teams (now New Zealand Institute for Bioeconomy Science Limited).


## Citations
> Chen T-H, Angelin-Bonnet O, Bristow J, Benson C, Ou S, Deng C, Thomson S. **TEgenomeSimulator: A Flexible Framework for Simulating Genomes with Configurable Transposable Element Landscapes.** bioRxiv; 2026. p. 2026.03.09.710711. https://doi.org/10.64898/2026.03.09.710711
- If you use TEgenomeSimulator v0.1.0, please cite it as:
  > **TEgenomeSimulator: A tool to simulate TE mutation and insertion into a random-synthesised or user-provided genome.** \
  > Ting-Hsuan Chen, Olivia Angeline-Bonnet, Cecilia Deng, Susan Thomson \
  > zenodo. 2024. doi: 10.5281/zenodo.14744027


## Contact person
Ting-Hsuan Chen: ting-hsuan.chen@plantandfood.co.nz

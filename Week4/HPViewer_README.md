# Detecting HPV in DNA-Sequencing Data with HPViewer

This beginner-friendly activity introduces students to bioinformatics by using **HPViewer** to detect and identify human papillomavirus (HPV) types in sequencing data.

## Learning goals

By the end of this activity, you will be able to:

- use basic Terminal commands;
- recognize FASTQ sequencing files;
- run a bioinformatics program;
- find the HPV type detected in a sample;
- interpret read count and RPK values.

> **Important:** This activity is for education and research only.

## What is HPViewer?

HPViewer is a command-line program that detects, types, and estimates the abundance of HPV in shotgun DNA-sequencing data. It maps sequencing reads directly to a collection of HPV reference genomes.

HPViewer reports:

- the detected HPV type;
- RPK, or HPV reads per kilobase of the effective reference sequence;
- the number of reads assigned to that HPV type.

## Which HPV reference does HPViewer use?

You do **not** need to download a separate `HPV16.fa` file.

The HPViewer repository already contains its required HPV reference databases in the `database/` directory. The original HPViewer study constructed these databases from **182 complete HPV reference genomes downloaded from PaVE (Papillomavirus Episteme)**.

HPViewer contains two customized databases:

1. **Repeat-mask database**
   - Low-complexity and simple-repeat regions are replaced with `N`.
   - This mode is designed to retain high sensitivity.

2. **Homology-mask database**
   - Regions highly similar among different HPV types are masked.
   - Simple-repeat regions are also masked.
   - This helps reduce incorrect assignment between closely related HPV types.

The default **hybrid-mask** mode first uses the repeat-mask database and then uses the homology-mask database to check closely related HPV types.

These are special multi-HPV reference databases created for HPViewer. A single PaVE HPV16 FASTA file used by another pipeline, such as searcHPV, is **not** a replacement for the HPViewer databases.

## What computer can I use?

The following instructions are written for a local Mac or Linux computer using Terminal. Windows students should use WSL or a Linux computer.

HPViewer was originally developed for a Linux-style command-line environment. The commands may also work on macOS, but Linux or WSL is the safest option if a compatibility problem occurs.

## Files needed

For the included demonstration, HPViewer provides:

```text
test_unpaired.fastq
```

For your own paired-end sample, you need two matching FASTQ files:

```text
sample_R1.fastq.gz
sample_R2.fastq.gz
```

FASTQ files contain sequencing-read names, DNA sequences, and base-quality scores.

## Step 1: Install Miniconda

If the `conda` command is already available, skip this step.

Download and install Miniconda from:

<https://docs.conda.io/projects/miniconda/en/latest/>

Close and reopen Terminal after installation. Then check:

```bash
conda --version
```

## Step 2: Create a software environment

Create an environment containing Python and the programs used by HPViewer:

```bash
conda create -n hpviewer \
  -c conda-forge \
  -c bioconda \
  python=3.10 \
  bowtie2 \
  samtools \
  bedtools \
  git \
  -y
```

Activate it:

```bash
conda activate hpviewer
```

Confirm that the programs are available:

```bash
python --version
bowtie2 --version
samtools --version
bedtools --version
```

> If `conda activate` produces an initialization error, run `conda init`, close Terminal, reopen it, and try again.

## Step 3: Download HPViewer

Move to a convenient location, such as your Documents folder:

```bash
cd ~/Documents
```

Download HPViewer:

```bash
git clone https://github.com/yuhanH/HPViewer.git
```

Enter the new folder:

```bash
cd HPViewer
```

Look at its contents:

```bash
ls
```

You should see `HPViewer.py`, `database`, and `test_unpaired.fastq`.

Check that the two HPV databases are present:

```bash
ls database/repeat-mask
ls database/homology-mask
```

## Step 4: Run the included test sample

Run HPViewer on the included single-end FASTQ file:

```bash
python HPViewer.py \
  -U test_unpaired.fastq \
  -o TEST \
  -p 2
```

The options mean:

| Option | Meaning |
|---|---|
| `-U` | Single-end or unpaired FASTQ file |
| `-o` | Output-directory name |
| `-p` | Number of computer threads |

When the command finishes, examine the output:

```bash
ls TEST
cat TEST/TEST_HPV_profile.txt
```

## Step 5: Understand the result

The main result file is:

```text
TEST/TEST_HPV_profile.txt
```

Its important columns are:

| Column | Meaning |
|---|---|
| HPV type | The HPV type detected in the sample |
| RPK | HPV-aligned reads normalized by effective reference length in kilobases |
| Count of reads | Number of sequencing reads assigned to that HPV type |

A larger read count or RPK means that more sequencing reads matched that HPV reference. It does not by itself measure disease severity.

HPViewer normally requires reads to cover more than a minimum amount of an HPV reference before reporting that type. The default threshold is 150 covered base pairs.

## Step 6: Run paired-end FASTQ files

Place `sample_R1.fastq.gz` and `sample_R2.fastq.gz` in the HPViewer folder, or provide their complete paths.

Run:

```bash
python HPViewer.py \
  -1 sample_R1.fastq.gz \
  -2 sample_R2.fastq.gz \
  -o sample_output \
  -p 4
```

View the profile:

```bash
cat sample_output/sample_output_HPV_profile.txt
```

The R1 and R2 files must belong to the same sample.

## Optional: Select a database mode

HPViewer uses hybrid-mask mode by default. The `-m` option must appear before the FASTQ options.

Default hybrid mode:

```bash
python HPViewer.py \
  -m hybrid-mask \
  -1 sample_R1.fastq.gz \
  -2 sample_R2.fastq.gz \
  -o sample_hybrid \
  -p 4
```

Repeat-mask mode:

```bash
python HPViewer.py \
  -m repeat-mask \
  -1 sample_R1.fastq.gz \
  -2 sample_R2.fastq.gz \
  -o sample_repeat \
  -p 4
```

Homology-mask mode:

```bash
python HPViewer.py \
  -m homology-mask \
  -1 sample_R1.fastq.gz \
  -2 sample_R2.fastq.gz \
  -o sample_homology \
  -p 4
```

For this introductory activity, keep the default hybrid-mask mode unless your instructor asks you to compare modes.

## Optional: Change the coverage threshold

The default minimum covered length is 150 bp. To require at least 200 bp, use:

```bash
python HPViewer.py \
  -1 sample_R1.fastq.gz \
  -2 sample_R2.fastq.gz \
  -o sample_c200 \
  -p 4 \
  -c 200
```

Do not change this threshold when comparing samples unless the same threshold is used for every sample.

## Common problems

### `command not found: conda`

Miniconda is not installed or Terminal has not been restarted after installation.

### `bowtie2 not found`

Activate the environment:

```bash
conda activate hpviewer
```

Then confirm:

```bash
which bowtie2
```

### `No such file or directory`

Check the current directory and filenames:

```bash
pwd
ls
```

Remember that capitalization matters: `sample_R1.fastq.gz` and `sample_r1.fastq.gz` are different names.

### The output directory already exists

Choose a new output name, such as:

```bash
-o sample_output_2
```

### No HPV type is reported

This may mean that no HPV reads passed the detection threshold. It does not prove that HPV is absent from the original biological sample because sequencing depth, sample quality, and analytical sensitivity also affect detection.

## Student questions

1. Which HPV type was detected in the test sample?
2. How many reads were assigned to it?
3. What does RPK mean?
4. Why does HPViewer mask repetitive DNA sequences?
5. Why might homologous regions cause one HPV type to be mistaken for another?
6. What is the difference between a sequencing read and a reference genome?
7. What happens if you raise the coverage threshold from 150 to 200 bp?

## Citation and resources

- HPViewer source code: <https://github.com/yuhanH/HPViewer>
- HPViewer paper: Hao Y. et al. *Bioinformatics* (2018), <https://doi.org/10.1093/bioinformatics/bty037>
- PaVE reference genomes: <https://pave.niaid.nih.gov/explore/reference_genomes/human_genomes>


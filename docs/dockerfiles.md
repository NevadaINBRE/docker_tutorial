# Building Custom Images (Dockerfiles)

While pulling existing images is useful, the true power of Docker lies in writing your own **Dockerfiles**. A Dockerfile is simply a text document containing all the commands a user could call on the command line to assemble an environment.

In this section, we will build a custom bioinformatics container capable of taking raw SARS-CoV-2 reads, cleaning them, aligning them to a reference genome, and generating a sorted BAM file.

## 1. Writing the Dockerfile

First, create a new directory for this project and open a blank file named exactly `Dockerfile` (no extension):

```bash
mkdir ~/covid_pipeline
cd ~/covid_pipeline
nano Dockerfile
```

Add the following "recipe" to your Dockerfile:

```bash
# Start from a stable, clean Ubuntu base
FROM ubuntu:22.04 # (1)!

# Prevent interactive prompts from halting the build
ENV DEBIAN_FRONTEND=noninteractive # (2)!

# Update the package manager and install our tools
RUN apt-get update && apt-get install -y \
    wget \
    fastqc \
    fastp \
    bwa \
    samtools \
    && rm -rf /var/lib/apt/lists/* # (3)!

# Set the default directory when the container starts
WORKDIR /data # (4)!
```

1. `FROM` defines the base operating system.
2. `ENV` sets environmental variables. Here, we tell `apt` not to ask us `[Y/n]` questions that would break an automated build.
3. `RUN` executes standard Linux commands inside the image during the build process. Notice how we clean up the downloaded package lists (`rm -rf`) in the exact same command; this is a strict Docker best practice to keep the final image size small!
4. `WORKDIR` ensures that any time we mount a volume or log into this container, we automatically start in the `/data` folder.

## 2. Building the Image

Now, tell Docker to build this recipe into a ready-to-use image. Run this command in the same folder as your Dockerfile:

```bash
docker build -t covid-pipeline:v1 .
```

!!! info "The Build Context"
    Don't forget the period (`.`) at the end of the command! It tells Docker to look in the current directory for the Dockerfile.

## 3. Preparing the Data

Let's download our specific SARS-CoV-2 dataset and reference genome to our host machine:

```bash
mkdir ~/covid_data
cd ~/covid_data

# Download the raw single-end COVID reads
wget https://biox.unr.edu/ftp/biox_microbiome_workshop/SRR19195566_covid_sra_data.fastq.gz

# Download the reference genome
wget http://ftp.ensemblgenomes.org/pub/viruses/fasta/sars_cov_2/dna/Sars_cov_2.ASM985889v3.dna.toplevel.fa.gz
```

## 4. Running the Pipeline

Now we mount our local `~/covid_data` directory into our custom `covid-pipeline:v1` container and run our tools. 

Because we set `WORKDIR /data` in the Dockerfile, the tools will automatically look in the mounted directory!

**Step 1: Index the Reference Genome**
```bash
docker run --rm -v ~/covid_data:/data covid-pipeline:v1 bwa index Sars_cov_2.ASM985889v3.dna.toplevel.fa.gz
```

**Step 2: Quality Control & Trimming**
```bash
docker run --rm -v ~/covid_data:/data covid-pipeline:v1 \
  fastp -i SRR19195566_covid_sra_data.fastq.gz \
        -o trimmed_reads.fastq.gz \
        -h fastp_report.html
```

**Step 3: Alignment & Sorting**
We can chain commands together using `bash -c` to align the trimmed reads with `bwa mem` and immediately pipe the output into `samtools` to generate a sorted BAM file.

??? tip "Try writing the command yourself first!"
    We need to mount the volume, call `bash -c`, run `bwa mem`, pipe (`|`) to `samtools view -bS`, and pipe to `samtools sort`.

    **Solution:**
    ```bash
    docker run --rm -v ~/covid_data:/data covid-pipeline:v1 bash -c \
      "bwa mem Sars_cov_2.ASM985889v3.dna.toplevel.fa.gz trimmed_reads.fastq.gz | \
       samtools view -bS - | \
       samtools sort -o covid_aligned.sorted.bam -"
    ```

**Step 4: Generate Mapping Statistics**
To see how well our reads aligned to the reference genome, we can use `samtools stats` on our newly sorted BAM file. This generates a text file that MultiQC will automatically detect and plot for us.

```bash
docker run --rm -v ~/covid_data:/data covid-pipeline:v1 bash -c \
  "samtools stats covid_aligned.sorted.bam > covid_aligned.stats"
```

Check your local folder. You successfully took raw sequencer output, trimmed it, aligned it to a viral genome, and generated mapping statistics, all without installing a single bioinformatics tool on your actual computer!

## 5. Other Package Managers: pip and Conda

Using `apt-get` is great for core system tools, but bioinformatics relies heavily on Python and Conda environments. You can absolutely use these package managers inside a Dockerfile.

Let's build two more containers and actually run them on the SARS-CoV-2 data we just generated!

### Example 1: Building a Python/pip Container

Since we ran `fastp` earlier, we generated quality control reports. Let's build a Python container with **MultiQC** to summarize those results.

Create a new directory and a new `Dockerfile`:
```bash
mkdir ~/multiqc_container
cd ~/multiqc_container
nano Dockerfile
```

Add this recipe:
```bash
# Start from an official, lightweight Python base image
FROM python:3.11-slim # (1)!

# Install Python packages using pip
RUN pip install --no-cache-dir multiqc # (2)!

# Set the working directory
WORKDIR /data

# Set the default command that runs if no command is provided
CMD ["multiqc", "--help"] # (3)!
```

1. Using `python:3.11-slim` gives you a pre-configured Python environment that is much smaller than a full Ubuntu image, saving download time.
2. `--no-cache-dir` is the `pip` equivalent of cleaning up `apt` lists. It forces pip to delete the downloaded package files after installation, drastically reducing your final image size.
3. `CMD` provides a default executable. If a user runs `docker run custom-multiqc` without specifying a command, it will automatically print the MultiQC help menu.

**Build and Run:**
Build the image, then mount our `covid_data` directory and run MultiQC. Because we told it to scan the current directory (`.`), it will automatically find *both* the `fastp` JSON log and the `samtools stats` file we generated!

```bash
# Build the image
docker build -t custom-multiqc:v1 .

# Run MultiQC on our existing data folder
docker run --rm -v ~/covid_data:/data custom-multiqc:v1 multiqc .
```

If you check your `~/covid_data` folder on your host machine, you will now see a beautiful `multiqc_report.html` file! Open it in your web browser to see an interactive dashboard showing both your read trimming quality and your viral genome alignment rates.

### Example 2: Building a Bioconda Container

Conda is incredibly popular in genomics because it handles messy C++ dependencies seamlessly. Let's use Miniconda to install **bedtools**, which we can use to convert our sorted BAM file into a BED file.

Create a new directory and a new `Dockerfile`:
```bash
mkdir ~/bedtools_container
cd ~/bedtools_container
nano Dockerfile
```

Add this recipe:
```bash
# Start from the official Miniconda image
FROM continuumio/miniconda3 # (1)!

# Set up the standard bioinformatics channels
RUN conda config --add channels defaults \
    && conda config --add channels bioconda \
    && conda config --add channels conda-forge # (2)!

# Install bedtools and clean up the cache in the same step
RUN conda install -y bedtools \
    && conda clean -a -y # (3)!

WORKDIR /data
```

1. `continuumio/miniconda3` comes with the Conda package manager pre-installed and ready to use immediately.
2. We configure the standard Conda channels, ensuring `conda-forge` and `bioconda` are available so it can find our specific bioinformatics software.
3. Just like with `apt` and `pip`, we use `conda clean -a -y` immediately after installation to strip out all the downloaded cache files and keep the container lightweight.

**Build and Run:**
Build the image, mount our data, and use `bedtools bamtobed` to extract the genomic coordinates from our aligned BAM file.

```bash
# Build the image
docker build -t custom-bedtools:v1 .

# Convert the BAM file to a BED file
docker run --rm -v ~/covid_data:/data custom-bedtools:v1 \
  bedtools bamtobed -i covid_aligned.sorted.bam > ~/covid_data/covid_aligned.bed
```

You now have a clean `covid_aligned.bed` file on your local machine, generated entirely by a custom Conda container!

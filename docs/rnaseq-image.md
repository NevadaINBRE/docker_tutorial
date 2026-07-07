# Building an RNASeq Pipeline Image

In the [Dockerfiles](dockerfiles.md) section, we built containers using `apt-get` and `pip` as package managers. For serious bioinformatics work, most of the tools we need live in [Bioconda](https://bioconda.github.io/). The best way to access those packages inside a Docker image is to build on a base that already has a fast Conda solver built in.

In this section we will build a single, self-contained image that holds every tool needed for a standard RNA-seq pipeline:

| Tool | Purpose | Version |
|---|---|---|
| **FastQC** | Raw read quality control | 0.12.1 |
| **fastp** | Adapter trimming & QC | 0.23.4 |
| **STAR** | Splice-aware alignment | 2.7.11a |
| **featureCounts** (subread) | Read quantification | 2.0.6 |
| **MultiQC** | Aggregated QC report | 1.21 |

This image will later be used in the [Snakemake workshop](https://nevadainbre.github.io/snakemake_tutorial/) as a drop-in replacement for per-rule Conda environments.

## 1. Choosing a Base Image

Rather than starting from a plain Ubuntu image and manually installing Conda, we use **`mambaorg/micromamba`**. This official image ships with [Micromamba](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html), a tiny standalone reimplementation of the Conda package manager that resolves environments significantly faster than classic `conda`. It is the recommended base for reproducible bioinformatics images.

## 2. Writing the Dockerfile

Create a new directory for this project and open a blank `Dockerfile`:

```bash
mkdir ~/rnaseq_image
cd ~/rnaseq_image
nano Dockerfile
```

Add the following recipe:

```bash
# Start from the official Micromamba image for fast Conda installs
FROM mambaorg/micromamba:1.5.8 # (1)!

# Copy the environment specification into the image
COPY env.yaml /tmp/env.yaml # (2)!

# Install all bioinformatics tools in the base environment
RUN micromamba install -y -n base -f /tmp/env.yaml && micromamba clean -a -y # (3)!

# Bake the conda bin directory into PATH so tools are found in any runtime
ENV PATH="/opt/conda/bin:$PATH" # (4)!

# Activate the base environment for every subsequent command
ARG MAMBA_DOCKERFILE_ACTIVATE=1 # (5)!

# Set the default working directory
WORKDIR /data # (6)!
```

1. Pinning the `micromamba` image version (`1.5.8`) ensures the same Conda solver is used on every build, preventing subtle differences in dependency resolution over time.
2. We copy a separate `env.yaml` file into the image instead of listing packages inline. This mirrors a real-world best practice: the environment specification is version-controlled separately from the build instructions.
3. `micromamba clean -a -y` removes all downloaded package archives and index caches from inside the image in the same `RUN` layer, keeping the final image size as small as possible.
4. The Micromamba entrypoint activates the conda environment when using `docker run`, but tools like Apptainer/Singularity bypass the entrypoint and call `bash` directly. Adding `/opt/conda/bin` to `PATH` here ensures tools are found regardless of how the container is launched.
5. Setting this build argument tells the Micromamba base image to automatically activate the `base` environment in every subsequent `RUN` step and in the final container shell.
6. Any mounted volume will land here by default, so tool commands can reference relative file paths without needing full paths.

Now create the environment file alongside the Dockerfile:

```bash
nano env.yaml
```

```yaml
channels:
  - bioconda
  - conda-forge
  - defaults
dependencies:
  - fastqc=0.12.1
  - fastp=0.23.4
  - star=2.7.11a
  - subread=2.0.6
  - multiqc=1.21
```

!!! tip "Why a separate `env.yaml`?"
    Keeping the package list in a standalone YAML file has two advantages. First, the file is human-readable and easy to update. Second, Docker's layer-caching system will **only** re-run the `micromamba install` step when `env.yaml` actually changes, saving minutes on incremental rebuilds.

## 3. Building the Image

Run the build from inside the `~/rnaseq_image` directory (where both `Dockerfile` and `env.yaml` live):

```bash
docker build -t rnaseq-pipeline:v1 .
```

!!! info "Expected build time"
    STAR is a large package (~500 MB uncompressed). The first build typically takes 5–10 minutes depending on your connection. Subsequent builds are fast because Docker caches each layer.

Watch for the final output line:

```
Successfully tagged rnaseq-pipeline:v1
```

Confirm the image appears in your local registry:

```bash
docker images rnaseq-pipeline
```

## 4. Testing the Image

Before using the image in a real pipeline, quickly verify that every tool is accessible and reports the expected version:

```bash
docker run --rm rnaseq-pipeline:v1 fastqc --version
docker run --rm rnaseq-pipeline:v1 fastp --version
docker run --rm rnaseq-pipeline:v1 STAR --version
docker run --rm rnaseq-pipeline:v1 featureCounts -v
docker run --rm rnaseq-pipeline:v1 multiqc --version
```

Each command should return a short version string with no error. If any command fails with `command not found`, check that your `env.yaml` spelling matches the exact Conda package names.

!!! warning "featureCounts reports its version to stderr"
    `featureCounts -v` writes the version string to standard error. If you want to capture it: `docker run --rm rnaseq-pipeline:v1 featureCounts -v 2>&1`

## 5. Pushing to Docker Hub

You now have a portable, version-locked image that encapsulates an entire RNA-seq software stack. Push it to Docker Hub now so it is available on any machine for collaborators and cluster environments. Use the same tagging and pushing steps from the previous [Sharing Images](dockerhub.md) section, substituting `rnaseq-pipeline` for the image name:

```bash
docker tag rnaseq-pipeline:v1 yourusername/rnaseq-pipeline:v1
docker push yourusername/rnaseq-pipeline:v1
```

Once the push completes, this image is ready to use in the [Snakemake workshop](https://nevadainbre.github.io/snakemake_tutorial/), where it replaces per-rule Conda environments via a single `container:` directive.

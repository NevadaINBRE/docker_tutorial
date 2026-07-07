# Instant Custom Containers (Seqera)

You now know how to build custom images using a `Dockerfile`, install tools via `apt`, `pip`, or `conda`, and push them to Docker Hub. 

However, building images locally takes time, especially when Conda has to solve complex dependency trees. What if you just want a container with `samtools`, `bwa`, and `multiqc` combined, and you want it *right now* without writing a Dockerfile?

Enter **Seqera Containers**.

## 1. What is Seqera Containers?

[Seqera Containers](https://seqera.io/containers/) is a free, web-based tool created by the developers of [Nextflow](https://www.nextflow.io/). It allows you to search for any combination of Conda, Bioconda, Conda-forge, or PyPI packages. 

When you select your tools, Seqera's backend instantly generates a fully reproducible Docker (or Apptainer/Singularity) image containing exactly those tools. Because they cache millions of combinations, the image is usually ready to pull in seconds.

## 2. Using the Web Interface

Let's say we want a single container that has both `bwa` for alignment and `samtools` for processing, just like the one we built manually earlier.

1. Open your web browser and navigate to [https://seqera.io/containers/](https://seqera.io/containers/).
2. In the search bar, type `bwa` and click **Add** next to the `bioconda::bwa` package.
3. Next, search for `samtools` and click **Add** next to the `bioconda::samtools` package.
4. Ensure the Container Settings are set to **Docker** and your machine's architecture (likely **linux/amd64** for most PCs/HPCs, or **linux/arm64** if you are on an Apple Silicon Mac).

On the right side of the screen, Seqera will instantly generate a `docker pull` command pointing to their custom registry (the `community.wave.seqera.io` URL).

## 3. Pulling and Running the Image

Copy the generated command from the Seqera website and run it in your terminal. It will look something like this (the unique hash at the end ensures strict version control):

```bash
docker pull community.wave.seqera.io/library/bwa_samtools:fdfc83664d470d06
```

Once downloaded, you can use it exactly like the images you built yourself! Let's test it by asking the container to print the `samtools` and `bwa` help menus:

```bash
docker run --rm community.wave.seqera.io/library/bwa_samtools:fdfc83664d470d06 samtools --version

docker run --rm community.wave.seqera.io/library/bwa_samtools:fdfc83664d470d06 bwa
```

## 4. Why This is a Game Changer

Seqera Containers drastically speeds up exploratory data analysis and pipeline development. 

* **No Dockerfiles:** You don't have to write, debug, or maintain a build recipe.
* **Instant Availability:** If someone else in the world has requested that exact combination of tools before, the image is instantly available from Seqera's cache.
* **Absolute Reproducibility:** The unique hash in the image URL is tied to the exact versions of the underlying Conda packages. If you use that URL in a published paper or a Nextflow script, anyone running it will get the exact same computational environment forever.

!!! tip "Pipeline Integration"
    If you write workflows in Nextflow, you don't even need to use the website! Nextflow's "Wave" feature can automatically query this service in the background and provision these custom containers on the fly just by reading the `conda` directives in your script.

# Pre-built Bioinformatics Tools

Up to this point, we have been using a bare-bones Ubuntu image and running standard Linux commands like `grep` and `awk`. But what if you want to run complex bioinformatics software? 

The beauty of Docker is that you almost never have to install tools from scratch. The global bioinformatics community has already containerized nearly every tool you can think of. 

## 1. The Biocontainers Ecosystem

Before you ever write a custom Dockerfile, you should check registries like [Docker Hub](https://hub.docker.com/) or the [Biocontainers](https://biocontainers.pro/) project. Biocontainers is a community-driven initiative that automatically builds Docker images for every piece of software uploaded to Bioconda.

If a tool exists in bioinformatics, there is a 99% chance a pre-built container is already waiting for you.

## 2. Running NCBI BLAST

Let's look at a practical example: **NCBI BLAST**. Installing BLAST locally can be a headache of dependencies and paths. With Docker, we just pull the official NCBI image and run it.

Let's use the `sample.fasta` file we created in the previous section. We are going to mount our data directory and ask the BLAST container to align our sequences against themselves.

Run this command in your terminal:

```bash
docker run --rm \
  -v ~/docker_data:/data \
  ncbi/blast \
  blastn -query /data/sample.fasta -subject /data/sample.fasta -outfmt 7
```

**Understanding the Command:**

* **`--rm`**: Cleans up the container after it finishes.
* **`-v ~/docker_data:/data`**: Mounts our local directory so the container can see our FASTA file.
* **`ncbi/blast`**: This is the official image published by the National Center for Biotechnology Information. Because we don't have it locally yet, Docker will automatically download it.
* **`blastn ...`**: The actual command we are passing into the container. We are running a nucleotide BLAST, using `sample.fasta` as both the query and the subject, and asking for tabular output (`-outfmt 7`).



## 3. Saving the Output

Just like we learned in the Volumes section, we can redirect the output of that command so it saves locally to our hard drive instead of just printing to the screen.

```bash
docker run --rm \
  -u $(id -u):$(id -g) \
  -v ~/docker_data:/data \
  ncbi/blast \
  bash -c "blastn -query /data/sample.fasta -subject /data/sample.fasta -outfmt 7 > /data/blast_results.txt"
```

*Note: We used the `-u $(id -u):$(id -g)` flag to ensure the resulting text file is owned by your user account, not the `root` user!*

Check your local folder:

```bash
cat ~/docker_data/blast_results.txt
```

You just downloaded and executed a complex, industry-standard bioinformatics alignment tool in seconds, without installing a single dependency on your computer. 

Now that you know how to use existing tools, it's time to learn how to package your own pipelines by building custom images.

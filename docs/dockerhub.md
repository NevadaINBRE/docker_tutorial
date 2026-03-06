# Sharing Images (Docker Hub)

Building custom containers locally is great for your own workflows, but the true magic of Docker is **reproducibility**. To ensure a collaborator or a peer reviewer evaluating your next manuscript can run your exact pipeline, you need to share your image.

The standard way to distribute containers is by pushing them to a public registry like [Docker Hub](https://hub.docker.com/).

## 1. Logging In

First, you need a free Docker Hub account. Once you have registered on their website, log into your account directly from your terminal:

```bash
docker login # (1)!
```

1. This will prompt you for your Docker Hub username and password. Your credentials will be securely saved on your host machine.

## 2. Tagging Your Image

Right now, our pipeline image is named `covid-pipeline:v1`. However, Docker Hub requires a very specific naming convention so it knows *whose* account the image belongs to. 

The required format is: `username/repository:tag`.

Let's apply a new tag to our local image so it includes your Docker Hub username.

!!! note "Replace `yourusername`!"
    Make sure to replace `yourusername` in the commands below with your actual Docker Hub username!

```bash
# Syntax: docker tag <local-image> <dockerhub-username>/<repo-name>:<tag>
docker tag covid-pipeline:v1 yourusername/covid-pipeline:v1 # (1)!
```

1. `docker tag` does not duplicate the heavy image data! It simply adds a new label that points to the exact same underlying environment, very similar to creating a symbolic link in Linux.

## 3. Pushing to Docker Hub

Now that the image is properly labeled with your username, you can push it to the cloud:

```bash
docker push yourusername/covid-pipeline:v1 # (1)!
```

1. Docker is highly efficient. It only uploads the *layers* of the image that don't already exist on Docker Hub. Since our image was based on `ubuntu:22.04`, Docker skips uploading the base Ubuntu system and only uploads the layer containing the bioinformatics tools we installed (`bwa`, `fastp`, `samtools`, etc.).

!!! warning "Crucial Security Check"
    Never push containers that contain raw genomic data, sensitive patient information (PHI/HIPAA data), or hardcoded passwords/API keys. Images on Docker Hub are public by default! Always keep your data separate from your software using Volumes.

## 4. Pulling Your Image Anywhere

Congratulations! Your viral genomics pipeline is now published to the world. 

You can log into any other machine in the world—a colleague's laptop, an AWS cloud instance, or the university HPC cluster—and run your exact pipeline with a single command:

```bash
docker run --rm -v ~/data:/data yourusername/covid-pipeline:v1 bwa mem ...
```

By publishing your environments alongside your code and data, you ensure your scientific research is 100% reproducible.

## 5. Cleaning Up

Now that the workshop is complete, it is good practice to clean up our environments so we don't leave unused files and images taking up hard drive space.

### Deleting the Docker Hub Repository
For security reasons, Docker does not allow you to delete a repository directly from the command line. To remove the test image we just pushed:

1. Log into [Docker Hub](https://hub.docker.com/) in your web browser.
2. Click on **Repositories** in the top navigation bar.
3. Click on your `covid-pipeline` repository.
4. Click the **Settings** tab near the top right.
5. Scroll down to the **Danger Zone** and click **Delete repository**.
6. Type the name of the repository to confirm and delete it.

### Cleaning Up Local Directories
Let's remove the dummy data, FASTQ files, and Dockerfiles we created on our host machine during the exercises.

!!! danger "Be careful with `rm -rf`!"
    Double-check these paths before hitting Enter to ensure you don't accidentally delete important files on your system.

```bash
cd ~
rm -rf ~/docker_data
rm -rf ~/covid_pipeline
rm -rf ~/covid_data
rm -rf ~/multiqc_container
rm -rf ~/bedtools_container
```

### Cleaning Up Docker Storage
Finally, let's tell Docker to delete all the stopped containers, dangling build caches, and downloaded images (like the heavy `ubuntu` and `miniconda` base images) to reclaim our hard drive space.

```bash
docker system prune -a --volumes # (1)!
```

1. `-a` removes all unused images, not just dangling ones. `--volumes` removes all unused anonymous volumes. Docker will ask you to type `y` to confirm before it wipes the data.

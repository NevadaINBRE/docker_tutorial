# Welcome to Nevada Bioinformatics Center - Docker Workshop

<img src="https://upload.wikimedia.org/wikipedia/commons/4/4e/Docker_%28container_engine%29_logo.svg" alt="Docker Logo" width="250" style="margin-bottom: 20px;">

Welcome! This interactive workshop is designed specifically for research scientists who want to modernize their computational workflows. 

As our field increasingly relies on High-Performance Computing (HPC) and cloud infrastructure, the ability to build, share, and deploy reproducible software environments is no longer just a "nice-to-have" skill, *it is essential*. 

## Why Docker?
Have you ever tried to run a colleague's pipeline, only to spend three days fighting with conflicting Python versions, missing C++ libraries, or incompatible R packages? 

Docker solves the "it works on my machine" problem permanently. By packaging your code, its dependencies, and the operating system itself into a single, portable **container**, you ensure your analysis runs exactly the same way on your local laptop, the university HPC cluster, or the cloud.

## What You Will Learn
In this workshop, you will follow along in your own terminal to master:

1. **Container Basics:** Pulling and running isolated Linux environments.
2. **Data Management:** Mounting your raw sequence data into containers to process it safely.
3. **Building Environments:** Writing Dockerfiles to install specific tools into your own custom images.
4. **Reproducibility:** Publishing your custom images to Docker Hub so your published papers have perfectly reproducible computational environments.

Let's get started by ensuring your system meets the [Prerequisites](prerequisites.md).

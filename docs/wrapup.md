# Workshop Wrap-up

Congratulations! You have successfully completed the Nevada Bioinformatics Center Docker Workshop. 

Transitioning from traditional software installation to a containerized workflow is one of the most significant leaps you can make as a computational researcher. It can feel intimidating at first, but you now possess the core skills to make your science universally reproducible.

## What You Have Achieved

Over the course of this workshop, you have learned how to:

* **Run Isolated Environments:** Pull and execute foundational Linux containers.
* **Manage Scientific Data:** Safely mount your raw data into ephemeral containers using Volumes (`-v`), and elegantly sidestep the root ownership trap.
* **Leverage the Community:** Find and run pre-built, industry-standard tools like NCBI BLAST directly from the Biocontainers ecosystem.
* **Build Custom Pipelines:** Write `Dockerfiles` to construct your own tailored environments using `apt`, `pip`, and `conda`.
* **Publish Your Work:** Push your custom images to Docker Hub so reviewers and collaborators can instantly replicate your computational environment.
* **Generate Instant Containers:** Use Seqera Containers to rapidly prototype reproducible environments without writing a single line of a Dockerfile.

## Why This Matters

The scientific computing landscape is shifting. Journals and granting agencies increasingly mandate strict computational reproducibility. By packaging your pipelines in Docker, you are no longer just sharing a script that *might* work on someone else's machine. You are sharing the exact operating system, dependencies, and tool versions used to generate your biological insights.

## Next Steps & Resources

As you begin applying these tools to your own research workflows, here are a few excellent resources to keep bookmarked:

* [Docker Official Documentation](https://docs.docker.com/) - The ultimate reference for Docker commands and Dockerfile syntax.
* [Biocontainers Registry](https://biocontainers.pro/) - Search for pre-built images of almost any bioinformatics tool in existence.
* [Seqera Containers](https://seqera.io/containers/) - The web portal for instantly provisioning custom Conda and PyPI environments.

Thank you for taking the time to modernize your computational workflows with us. We can't wait to see the incredible, reproducible science you publish next!

*- Hans and the Nevada Bioinformatics Center Team*

---

## Cleaning Up

Now that the workshop is complete, it is good practice to clean up the local directories and Docker resources we created during the exercises.

### Deleting Docker Hub Repositories

For security reasons, Docker does not allow you to delete a repository directly from the command line. To remove any test images you pushed during the workshop:

1. Log into [Docker Hub](https://hub.docker.com/) in your web browser.
2. Click on **Repositories** in the top navigation bar.
3. Click on the repository you want to remove.
4. Click the **Settings** tab near the top right.
5. Scroll down to the **Danger Zone** and click **Delete repository**.
6. Type the name of the repository to confirm and delete it.

### Cleaning Up Local Directories

!!! danger "Be careful with `rm -rf`!"
    Double-check these paths before hitting Enter to ensure you do not accidentally delete important files on your system.

```bash
cd ~
rm -rf ~/docker_data
rm -rf ~/covid_pipeline
rm -rf ~/covid_data
rm -rf ~/multiqc_container
rm -rf ~/bedtools_container
rm -rf ~/rnaseq_image
```

### Cleaning Up Docker Storage

Remove all stopped containers, dangling build caches, and downloaded images to reclaim hard drive space:

```bash
docker system prune -a --volumes # (1)!
```

1. `-a` removes all unused images, not just dangling ones. `--volumes` removes all unused anonymous volumes. Docker will ask you to type `y` to confirm before it wipes the data.

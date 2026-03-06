# Your First Container

Now that Docker is installed, let's spin up a completely clean, isolated Linux environment. 

For bioinformatics and scientific computing, we frequently rely on standard Linux distributions to ensure our tools run the same way everywhere. Today, we will use Ubuntu.

## 1. Pulling the Image

Before we can run a container, we need its blueprint (called an **image**). By default, Docker looks for these images on Docker Hub. 

To download the official Ubuntu image to your local machine, run:

```bash
docker pull ubuntu
```

*Note: If you don't specify a version (like `ubuntu:22.04`), Docker automatically pulls the `latest` tag.*

## 2. Running Interactively

In standard HPC environments, you usually submit a script to a queue or request an interactive node. In Docker, you use the `docker run` command. 

Let's start an interactive session inside a new Ubuntu container:

```bash
docker run \
  -i \
  -t \
  --name my_ubuntu \
  ubuntu \
  /bin/bash
```

**Understanding the Command:**

* **`-i`**: Stands for **Interactive**. It keeps the standard input open even if not attached.
* **`-t`**: Allocates a **pseudo-TTY** (a terminal session). Together, `-it` is the standard way to drop into a container's shell.
* **`--name`**: Assigns a human-readable name to your container. If you skip this, Docker assigns a random, goofy name.
* **`ubuntu`**: The image we are building the container from.
* **`/bin/bash`**: The specific process we want to run inside the container.

## 3. Exploring the Container

If the command was successful, your terminal prompt should have changed. It will look something like this: `root@a1b2c3d4e5f6:/#`.

You are now the root user inside an isolated Linux environment! Prove to yourself that you are no longer on your host machine by checking the OS release:

```bash
cat /etc/os-release
```

You will see output confirming you are running Ubuntu, regardless of whether your actual laptop is a Mac, Windows, or a different Linux flavor. 

## 4. Exiting and Cleanup

To leave the container, simply type:

```bash
exit
```

Your prompt will return to your host machine.

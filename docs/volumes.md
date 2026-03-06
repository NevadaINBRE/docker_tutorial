# Data and Volumes

By design, Docker containers are ephemeral. When a container stops or is removed, any data generated inside it is destroyed. 

To do actual scientific computing, we need a way to pass our raw data into the container and save our results back out to our host machine. We do this using **Volumes**.

## 1. Creating Test Data

First, let's create a dummy directory and a sample FASTA file on your local machine to represent our genomic data. Run these commands in your host terminal:

```bash
mkdir -p ~/docker_data
cd ~/docker_data
cat << 'EOF' > sample.fasta
>seq1
ATGCGTACGTTAG
>seq2
CGTACGTAGCTAG
EOF
```

## 2. Mounting a Volume

To link this local directory to a directory inside the container, we use the `-v` (volume) flag. The syntax is always `host_path:container_path`.

Let's run a new container, mount our data, and see if the container can read it:

```bash
docker run --rm \
  -v ~/docker_data:/data \
  ubuntu \
  ls -l /data
```

**Understanding the Command:**

* **`--rm`**: Automatically deletes the container the moment it finishes running to keep your system clean.
* **`-v`**: Mounts the `~/docker_data` directory from your computer to a new folder called `/data` inside the container.
* **`ubuntu`**: The image we are running.
* **`ls -l /data`**: The command executed inside the container.

You should see `sample.fasta` listed in the output!

## 3. Processing and Saving Results

Now for the real test. Let's use the container to count the sequences in our FASTA file using `grep`, and write the output to a new file.

```bash
docker run --rm -v ~/docker_data:/data ubuntu bash -c "grep -c '^>' /data/sample.fasta > /data/counts.txt"
```

Once that container finishes and deletes itself, check your local `~/docker_data` directory:

```bash
cat ~/docker_data/counts.txt
```

You should see the number `2`. The container read your local data, processed it, and saved the result back to your hard drive before disappearing!

## 4. Filtering Data with `awk`

Containers are incredibly useful for running standard Unix tools to filter files for specific values without cluttering your host system. Here, we use `awk` to parse our FASTA file and print only the sequence identifiers:

```bash
docker run --rm \
  -v ~/docker_data:/data \
  ubuntu \
  awk '/^>/ {print $1}' /data/sample.fasta
```

Just like a local HPC node, the container rapidly processes the file and prints `>seq1` and `>seq2` straight to your terminal before instantly cleaning itself up.

## 5. The Root Ownership Trap

If you run containers that generate output files (like the `counts.txt` file we made earlier) and later try to delete or modify them on your host machine, you might get a frustrating `Permission denied` error. 

**Why does this happen?**
By default, the Docker daemon and the processes inside your containers run as the `root` user. When you mount a volume and the container writes a new file to your host's hard drive, it writes that file as `root`. Since your normal user account on your laptop or the HPC isn't root, you are locked out of your own files!

**The Fix:**
You can tell Docker to run the container matching your exact host User ID (UID) and Group ID (GID) using the `-u` flag. In Linux and macOS, you can dynamically fetch these using `$(id -u):$(id -g)`.

Let's test it by creating a new file, but this time ensuring we retain ownership:

```bash
docker run --rm \
  -u $(id -u):$(id -g) \
  -v ~/docker_data:/data \
  ubuntu \
  bash -c "echo 'I own this file' > /data/my_user_file.txt"
```

*Note: The `-u $(id -u):$(id -g)` flag maps your host account's identity into the container. Everything the container creates will now belong to you, not root!*

Now check the permissions on your host machine:

```bash
ls -l ~/docker_data/my_user_file.txt
```

You will see your own username listed as the owner instead of `root`, meaning you can freely edit, move, or delete it without needing `sudo`. 

!!! tip "Best Practice"
    Always use the `-u $(id -u):$(id -g)` flag when running bioinformatics pipelines that write large output files (like BAMs, VCFs, or MultiQC reports) back to your host system!

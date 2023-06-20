# An opinionated template for NLP research code

[![Docker Hub](https://img.shields.io/docker/v/konstantinjdobler/nlp-research-template/latest?color=blue&label=docker&logo=docker)](https://hub.docker.com/r/konstantinjdobler/nlp-research-template/tags) [![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black) ![License: MIT](https://img.shields.io/github/license/konstantinjdobler/nlp-research-template?color=green)

NLP research template for training language models from scratch using PyTorch + PyTorch Lightning + Weights & Biases + HuggingFace. It's built to be customized but provides comprehensive, sensible default functionality.

## Setup

### Preliminaries

It's recommended to use [`mamba`](https://github.com/mamba-org/mamba) to manage dependencies. `mamba` is a drop-in replacement for `conda` re-written in C++ to speed things up significantly (you can stick with `conda` though). To provide reproducible environments, we use `conda-lock` to generate lockfiles for each platform.

<details><summary>Installing <code>mamba</code></summary>

<p>

On Unix-like platforms, run the snippet below. Otherwise, visit the [mambaforge repo](https://github.com/conda-forge/miniforge#mambaforge). Note this does not use the Anaconda installer, which reduces bloat.

```bash
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh"
bash Mambaforge-$(uname)-$(uname -m).sh
```

</details>

<details><summary>Installing <code>conda-lock</code></summary>

<p>

The preferred method is to install `conda-lock` using `pipx install conda-lock`. For other options, visit the [conda-lock repo](https://github.com/conda/conda-lock). For basic usage, have a look at the commands below:

```bash
conda-lock install --name gpt5 conda-lock.yml # create environment with name gpt5 based on lockfile
conda-lock # create new lockfile based on environment.yml
conda-lock --update <package-name> # update specific packages in lockfile
```

</details>

### Environment

Lockfiles are an easy way to **exactly** reproduce an environment.

After having installed `mamba` and `conda-lock`, you can create a `mamba` environment named `gpt5` from a lockfile with all necessary dependencies installed like this:

```bash
conda-lock install --name gpt5 conda-lock.yml
```

You can then activate your environment with

```bash
mamba activate gpt5
```

To generate new lockfiles after updating the `environment.yml` file, simply run `conda-lock` in the directory with your `environment.yml` file.

For more advanced usage of environments (e.g. updating or removing environments) have a look at the [conda-documentation](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#removing-an-environment).

<details><summary>Setup on <code>ppc64le</code></summary>

<p>

**If you're not using a PowerPC machine, do not worry about this.**

Whenever you create an environment for a different processor architecture, some packages (especially `pytorch`) need to be compiled specifically for that architecture. IBM PowerPC machines for example use a processor architecture called <code>ppc64le</code>.
Setting up the environment <code>ppc64le</code> is a bit tricky because the official channels do not provide packages compiled for <code>ppc64le</code>. However, we can use the amazing [Open-CE channel](https://ftp.osuosl.org/pub/open-ce/current/) instead. A lockfile containing the relevant dependencies is already prepared in <code>ppc64le.conda-lock.yml</code> and the environment again can be simply installed with:

```bash
conda-lock install --name gpt5-ppc64le ppc64le.conda-lock.yml
```

Dependencies for <code>ppce64le</code> should go into the seperate <code>ppc64le.environment.yml</code> file. Use the following command to generate a new lockfile after updating the dependencies:

```bash
conda-lock --file ppc64le.environment.yml --lockfile ppc64le.conda-lock.yml
```

</p>
</details>

### Docker

For fully reproducible environments and running on HPC clusters, we provide pre-built docker images at [konstantinjdobler/nlp-research-template](https://hub.docker.com/r/konstantinjdobler/nlp-research-template/tags). We also provide a `Dockerfile` that allows you to build new docker images with updated dependencies:

```bash
docker build --tag <username>/<imagename>:<tag> --platform=linux/amd64 .
```

The specified username should be your personal [`dockerhub`](https://hub.docker.com) username. This will make distribution and reusage of your images easier.

## Development

Development for ML can be quite resource intensive. If possible, you can start your development on a more powerful host machine to which you connect to from your local machine. Normally, you would set up the correct environment on the host machine as explained above but this workflow is simplified a lot by using `VS Code Dev Containers`. They allow you to develop inside a docker container with all necessary dependencies pre-installed. The template already contains a `.devcontainer` directory, where all the settings for it are stored - you can start right away!

<details><summary>VS Code example</summary>

<p>

After having installed the [Remote-SSH-](https://code.visualstudio.com/docs/remote/ssh), and [Dev Containers-Extension](https://code.visualstudio.com/docs/devcontainers/containers), you set up your `Dev Container` in the following way:

1. Establish the SSH-connection with the host by opening your VS Code command pallet and typing <code>Remote-SSH: Connect to Host</code>. Now you can connect to your host machine.
2. Open the folder that contains this template on the host machine.
3. VS Code will automatically detect the `.devcontainer` directory and ask you to reopen the folder in a Dev Container.
4. Press <code>Reopen in Container</code> and wait for VS Code to set everything up.

When using this workflow you will have to adapt `"runArgs": ["--ipc=host", "--gpus", "device=CHANGE_ME"]` in [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json) and specify the GPU-devices you are actually going to use on the host machine for your development. Optionally you can mount cache files with `"mounts": ["source=/MY_HOME_DIR/.cache,target=/home/mamba/.cache,type=bind"]`. `conda-lock` is automatically installed for you in the Dev Container.

Additionally, you can set the `WANDB_API_KEY` in your remote environment; it will then be automatically mapped into the container.

</p>
</details>

## Training

After all of this setup you are finally ready for some training. First of all, you need to create your data directory with a `train.txt` and `dev.txt`. Then you can start a training run in your environment with:

```bash
python train.py -n <run-name> -d /path/to/data --model roberta-base --offline
```

To see an overview over all options and their defaults, run `python train.py --help`. We have disabled Weights & Biases syncing with the `--offline` flag. If you want to log your results, enable W&B as described [here](#weights--biases) and omit the `--offline` flag.

<details><summary>Using GPUs for hardware acceleration</summary>

<p>

By default, `train.py` already detects all available CUDA GPUs and uses `DistributedDataParallel` training in case multiple GPUs are found. You can manually select specific GPUs with `--cuda_device_ids`. To use different hardware accelerators, use the `--accelerator` flag. You can use advanced parallel training strategies with `--distributed_strategy`.

</p>
</details>

### Using the Docker environment for training

To run the training code inside the docker environment, start your container by executing the [console.sh](./scripts/console.sh) script. Inside the container you can now execute your training script as before.

```bash
bash ./scripts/console.sh   # use this to start the container
python train.py -n <run-name> -d /path/to/data/ --model roberta-base --offline # execute the training inside your container
```

Like when using a [`Dev Container`](#development), by default no GPUs are available inside the container and caches written to `~/.cache` inside the container will not be persistent. You can modify the [console.sh](./scripts/console.sh) script to select GPUs for training, a persistent cache directory and the docker image for the container. Also, make sure to mount the data directory into the container.

**Docker + GPUs:** Always select specififc GPUs via `docker` (e.g. `--gpus device=0,7` for the GPUs with indices `0` and `7` in [console.sh](./scripts/console.sh)) and set the `train.py` script to use all available GPUs for training with `--num_devices=-1` (which is the default).

**Note:** In order to mount a directory for caching you need to have one created first.

<details><summary>Single-line docker command</summary>

<p>

You can start a script inside a docker container in a single command (caches are not persistent in this example):

```bash
docker run -it --user $(id -u):$(id -g) --ipc host -v "$(pwd)":/workspace -w /workspace --gpus device=0,7 konstantinjdobler/nlp-research-template:latest python train.py --num_devices=-1 ...
```

</p>
</details>

<details><summary>Using Docker with SLURM / <code>pyxis</code></summary>

<p>

For security reasons, `docker` might be disabled on your HPC cluster. You might be able to use the SLURM plugin `pyxis` instead like this:

```bash
srun ... --container-image konstantinjdobler/nlp-research-template:latest python train.py ...
```

This uses [`enroot`](https://github.com/NVIDIA/enroot) under the hood to import your docker image and run your code inside the container. See the [`pyxis` documentation](https://github.com/NVIDIA/pyxis) for more options, such as `--container-mounts` or `--container-writable`.

If you want to run an interactive session with bash don't forget the `--pty` flag, otherwise the environment won't be activated properly.

</p>
</details>

### Weights & Biases

Weights & Biases is a platform that provides an easy way to log training results for ML researchers. It lets you create checkpoints of your best models, can save the hyperparameters of your model and even supports Sweeps for hyperparameter optimization. For more information you can visit the [website](https://wandb.ai/site). To enable Weights & Biases, enter your `WANDB_ENTITY` and `WANDB_PROJECT` in [dlib/frameworks/wandb.py](dlib/frameworks/wandb.py) and omit the `--offline` flag for training.

<details><summary>Weights & Biases + Docker</summary>

<p>

When using docker you also have to provide your `WANDB_API_KEY`. You can find your personal key at [wandb.ai/authorize](https://app.wandb.ai/authorize). Either set `WANDB_API_KEY` on your host machine and use the `docker` flag `--env WANDB_API_KEY` when starting your run or use `wandb docker-run` instead of docker run.

</p>
</details>

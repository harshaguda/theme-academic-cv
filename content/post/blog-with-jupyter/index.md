---
title: Isaac Orbit on Triton HPC
date: '2023-11-13'
summary: Running Isaac Orbit experiments on High Performance clusters using docker image.
---

# Isaac Orbit on Triton

## Building Isaac Orbit Docker Image

Follow the instructions on setting up docker and nvidia docker images on local machine from the following [link.](https://docs.omniverse.nvidia.com/isaacsim/latest/install_container.html)

Create the following `dockerfile` 

Ref: https://github.com/NVIDIA-Omniverse/Orbit/discussions/14

```docker
ARG ISAACSIM_VERSION=2022.2.1
FROM nvcr.io/nvidia/isaac-sim:${ISAACSIM_VERSION}

### Use bash by default
SHELL ["/bin/bash", "-c"]

### Install Orbit
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
        cmake \
        ncurses-term \
        git && \
    rm -rf /var/lib/apt/lists/* && \
    git clone https://github.com/NVIDIA-Omniverse/Orbit.git --depth 1 -b main /orbit && \
    ln -s "/isaac-sim" /orbit/_isaac_sim && \
    /orbit/orbit.sh --install && \
    /orbit/orbit.sh --extra || exit 0

### Set entrypoint (adjust to your preferences)
ENTRYPOINT ["/orbit/orbit.sh", "-p"]
```

A docker image with above configurations is built and can be downloaded using the command below.

If you are planning to create your own environments, I used the following work around might not be an efficient way but works.

Copy the `/orbit` folder from docker on to your triton workspace. If you have created new environments copy them using to the contrib folder using the following command. (Read more about [docker cp](https://docs.docker.com/engine/reference/commandline/cp/) and [scp](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/?utm_content=cmp-true)).

```bash
cp -rf ${WRKDIR}/summer-project/Orbit/source/extensions/omni.isaac.contrib_envs/omni/isaac/contrib_envs/<env_folder_name> ${WRKDIR}/orbit/source/extensions/omni.isaac.contrib_envs/omni/isaac/contrib_envs/
```

```bash
cd $WRKDIR
singularity build orbit-isaac-2022-2-1.sif docker://harshavguda/orbit-isaacsim:2022.2.1
singularity build orbit-isaac-2022-2-1.sif <name_of_container>

```

Copy the created `.sif` file to triton.

To open shell of the docker image run the following command. 

```bash
srun singularity shell -B /m:/m -B /l:/l -B /scratch:/scratch -B /etc/vulkan/icd.d/nvidia_icd.json -B /etc/vulkan/implicit_layer.d/nvidia_layers.json -B /usr/share/glvnd/egl_vendor.d/10_nvidia.json -B ${WRKDIR}/isaac-sim/kit/cache/Kit:/isaac-sim/kit/cache/Kit,${WRKDIR}/isaac-sim/cache/ov:/isaac-sim/cache/ov,${WRKDIR}/isaac-sim/cache/pip:/isaac-sim/cache/pip,${WRKDIR}/isaac-sim/cache/glcache:/isaac-sim/cache/glcache,${WRKDIR}/isaac-sim/cache/computecache:/isaac-sim/cache/computecache,${WRKDIR}/isaac-sim/logs:/isaac-sim/logs,${WRKDIR}/isaac-sim/data:/isaac-sim/data,${WRKDIR}/isaac-sim/documents:/isaac-sim/documents --nv ${WRKDIR}/orbit-isaac-2022-2-1.sif
```

## Examples

Example script run a custom environment.

```bash
#!/bin/bash -l
#SBATCH --gres=gpu:1
#SBATCH --time=02:00:50
#SBATCH --mem=16G
#SBATCH --cpus-per-task=6
#SBATCH --constraint=ampere
export ACCEPT_EULA=Y
export ISAACSIM_PATH="/isaac-sim"
export ISAACSIM_PYTHON_EXE="/isaac-sim/python.sh"
cp -rf ${WRKDIR}/summer-project/Orbit/source/extensions/omni.isaac.contrib_envs/omni/isaac/contrib_envs/bagmanipulation/ ${WRKDIR}/orbit/source/extensions/omni.isaac.contrib_envs/omni/isaac/contrib_envs/
cp -rf ${WRKDIR}/summer-project/Orbit/source/extensions/omni.isaac.contrib_envs/omni/isaac/contrib_envs/bagopening/ ${WRKDIR}/orbit/source/extensions/omni.isaac.contrib_envs/omni/isaac/contrib_envs/

# Run bag lift experiment
srun  singularity exec -B /m:/m -B /l:/l -B /scratch:/scratch -B /etc/vulkan/icd.d/nvidia_icd.json -B /etc/vulkan/implicit_layer.d/nvidia_layers.json -B /usr/share/glvnd/egl_vendor.d/10_nvidia.json -B /scratch/work/gudah1/isaac-sim/kit/cache/Kit:/isaac-sim/kit/cache/Kit,/scratch/work/gudah1/isaac-sim/cache/ov:/isaac-sim/cache/ov,/scratch/work/gudah1/isaac-sim/cache/pip:/isaac-sim/cache/pip,/scratch/work/gudah1/isaac-sim/cache/glcache:/isaac-sim/cache/glcache,/scratch/work/gudah1/isaac-sim/cache/computecache:/isaac-sim/cache/computecache,/scratch/work/gudah1/isaac-sim/logs:/isaac-sim/logs,/scratch/work/gudah1/isaac-sim/data:/isaac-sim/data,/scratch/work/gudah1/isaac-sim/documents:/isaac-sim/documents --nv ${WRKDIR}/orbit-isaac-2022-2-1.sif <exec_command>
```
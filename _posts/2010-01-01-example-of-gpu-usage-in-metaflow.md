---
date: 2020-07-21
layout: post
title: Example of GPU usage in Metaflow
subtitle:
description: GPU usage for deep learning is not trivial in Metaflow due to the fact that one needs to install CUDA in a correct way and expose the GPU hardwares to the container that runs Metaflow steps.
image: /blog-theme/assets/img/posts/example-of-gpu-usage-in-metaflow/metaflow.jpg
optimized_image: /blog-theme/assets/img/posts/example-of-gpu-usage-in-metaflow/metaflow.jpg
category: life
tags:
  - python
authors: wesleytanner, kellygajewski
paginate: true
---

The `test_metaflow_gpu.py` contains an example of how to allocate
GPU resources and make use of it for deep learning in Metaflow.

GPU usage for deep learning is not trivial in Metaflow due to
the fact that one needs to install CUDA in a correct way and
expose the GPU hardwares to the container that runs Metaflow steps.

Core setup (in Metaflow) is to use the `resource(gpu=N)` decorator
for EC2 resources and the `batch(..., image=IMAGE_WITH_CUDA)` decorator
(or the use no image and set up CUDA/NVIDIA environment variables with
[Metaflow(>=2.2.8)](https://docs.metaflow.org/introduction/release-notes#2-2-8-mar-15th-2021)
`@environment` decorator) for AWS Batch resources

```python
@batch(
    cpu=2,
    gpu=2,
    memory=2400,
    # This image can work without the @environment decorator because NVIDIA and CUDA related environment
    # variables have been set up in the docker
    # image="763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.7.1-gpu-py36-cu110-ubuntu18.04"
)
@environment(
    # environment decorator is used to set up global system level
    # environment variables
    vars={
        'NVIDIA_DRIVER_CAPABILITIES': 'compute,utility',  # necessary to allow computation on GPUs
        'CUDA_VISIBLE_DEVICES': '0,1',  # make two CUDA devices visible
    }
)
@conda(libraries={
    'pytorch::pytorch': '1.7.1',  # need this conda decorator to install pytorch.
    # Metaflow will not automatically recognize the pytorch installed in the above image
    # because it will create its own conda env
    # The first pytorch before '::' is the pytorch conda channel
    # One wants to install from that channel for CUDA support
    # The conda-forge channel's pytorch doesn't come with CUDA support
})
def start(self):
    ...
```

The image `763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.6.0-gpu-py36-cu110-ubuntu18.04` used above can be obtained from [Amazon Deep Learning Images](https://github.com/aws/deep-learning-containers/blob/master/available_images.md) project. A variety of deep learning images with different types of deep learning frameworks (Tensorflow, Pytorch, etc), different CUDA versions (CUDA10, CUDA11) are provided.

## How to run

```shell
USERNAME=YOURUSERNAME CONDA_CHANNELS=default,pytorch,conda-forge METAFLOW_PROFILE=ds-dev AWS_PROFILE=ds-dev-admin python test_metaflow_gpu.py --environment=conda run
```

For how to setup the `METAFLOW_PROFILE` and `AWS_PROFILE`, refer to the [local_dev](../../local_dev) folder.

Expected result should be

<details>
    <summary>Expected output (click to expand)</summary>

```shell
(base) root@2b15d8a4c149:/ds-model-mlm-metaflow/tests/test_metaflow_gpu# USERNAME=bz CONDA_CHANNELS=default,pytorch,conda-forge METAFLOW_PROFILE=ds-dev AWS_PROFILE=ds-dev-admin python test_metaflow_gpu.py --environment=conda run
Metaflow 2.2.7 executing TestGPUFlow for user:bz
Validating your flow...
    The graph looks good!
Running pylint...
    Pylint is happy!
Bootstrapping conda environment...(this could take a few minutes)
2021-03-12 04:51:56.446 Workflow starting (run-id 129):
2021-03-12 04:52:00.209 [129/start/1341 (pid 5161)] Task is starting.
2021-03-12 04:52:02.427 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status SUBMITTED)...
2021-03-12 04:52:04.723 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status RUNNABLE)...
2021-03-12 04:52:34.796 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status RUNNABLE)...
2021-03-12 04:53:04.927 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status RUNNABLE)...
2021-03-12 04:53:35.036 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status RUNNABLE)...
2021-03-12 04:54:05.230 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status RUNNABLE)...
2021-03-12 04:54:35.386 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status RUNNABLE)...
2021-03-12 04:54:45.615 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status STARTING)...
2021-03-12 04:55:15.622 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status STARTING)...
2021-03-12 04:55:45.627 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status STARTING)...
2021-03-12 04:56:15.788 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status STARTING)...
2021-03-12 04:56:45.927 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status STARTING)...
2021-03-12 04:57:16.127 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status STARTING)...
2021-03-12 04:57:46.147 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status STARTING)...
2021-03-12 04:58:15.881 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting (status RUNNING)...
2021-03-12 04:58:20.154 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] bash: cannot set terminal process group (-1): Inappropriate ioctl for device
2021-03-12 04:58:20.155 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] bash: no job control in this shell
2021-03-12 04:58:20.156 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] AWS_BATCH_CE_NAME='MetaflowComputeEnvironment'
2021-03-12 04:58:20.156 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] AWS_BATCH_JOB_ATTEMPT='1'
2021-03-12 04:58:20.157 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] AWS_BATCH_JOB_ID='931bbcf6-5fa2-4661-83df-1d82d52b6226'
2021-03-12 04:58:20.158 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] AWS_BATCH_JQ_NAME='job-queue-metaflow-infrastructure'
2021-03-12 04:58:20.158 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] AWS_CONTAINER_CREDENTIALS_RELATIVE_URI='/v2/credentials/0a3ac1ce-0a34-41bc-a916-e2cf51a907c0'
2021-03-12 04:58:20.159 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] AWS_DEFAULT_REGION='us-east-1'
2021-03-12 04:58:20.160 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] AWS_EXECUTION_ENV='AWS_ECS_EC2'
2021-03-12 04:58:20.160 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] CMAKE_PREFIX_PATH='$(dirname $(which conda))/../'
2021-03-12 04:58:20.161 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] CUDA_VERSION='11.0.3'
2021-03-12 04:58:20.161 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] CUDNN_VERSION='8.0.4.30'
2021-03-12 04:58:20.162 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] DGLBACKEND='pytorch'
2021-03-12 04:58:20.162 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] ECS_CONTAINER_METADATA_URI='http://169.254.170.2/v3/615eaa62-7e66-46b8-9b7f-feafa2874358'
2021-03-12 04:58:20.162 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] ECS_CONTAINER_METADATA_URI_V4='http://169.254.170.2/v4/615eaa62-7e66-46b8-9b7f-feafa2874358'
2021-03-12 04:58:20.163 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] HOME='/root'
2021-03-12 04:58:20.163 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] HOROVOD_VERSION='0.20.3'
2021-03-12 04:58:20.164 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] HOSTNAME='ip-10-13-90-128.ec2.internal'
2021-03-12 04:58:20.164 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] IFS='
2021-03-12 04:58:20.165 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] '
2021-03-12 04:58:22.935 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] LANG='C.UTF-8'
2021-03-12 04:58:22.936 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] LC_ALL='C.UTF-8'
2021-03-12 04:58:22.937 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] LD_LIBRARY_PATH='/opt/conda/lib/python3.6/site-packages/smdistributed/dataparallel/lib:/home/.openmpi/lib/:/opt/conda/lib:/usr/local/lib:/usr/local/nvidia/lib:/usr/local/nvidia/lib64'
2021-03-12 04:58:22.938 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] LIBRARY_PATH='/usr/local/cuda/lib64/stubs'
2021-03-12 04:58:22.939 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] MANUAL_BUILD='0'
2021-03-12 04:58:22.940 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_CODE_DS='s3'
2021-03-12 04:58:22.940 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_CODE_SHA='0cade183b6e658b8d9b0343997eace8b235334cf'
2021-03-12 04:58:22.941 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_CODE_URL='s3://zefr-ds-dev-use1-865426939207-s3-metaflow/metaflow/TestGPUFlow/data/0c/0cade183b6e658b8d9b0343997eace8b235334cf'
2021-03-12 04:58:22.942 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_DATASTORE_SYSROOT_S3='s3://zefr-ds-dev-use1-865426939207-s3-metaflow/metaflow'
2021-03-12 04:58:22.943 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_DATATOOLS_S3ROOT='s3://zefr-ds-dev-use1-865426939207-s3-metaflow/metaflow/data'
2021-03-12 04:58:22.944 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_DEFAULT_DATASTORE='s3'
2021-03-12 04:58:22.944 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_DEFAULT_METADATA='service'
2021-03-12 04:58:22.945 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_INPUT_PATHS_0='129/_parameters/1340'
2021-03-12 04:58:22.946 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_SERVICE_HEADERS='{}'
2021-03-12 04:58:22.948 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_SERVICE_URL='http://tf-lb-20210225192300834700000001-bb0309f275885bf4.elb.us-east-1.amazonaws.com/'
2021-03-12 04:58:22.949 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] METAFLOW_USER='bz'
2021-03-12 04:58:22.950 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] NCCL_VERSION='2.7.8'
2021-03-12 04:58:22.951 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] NVIDIA_DRIVER_CAPABILITIES='compute,utility'
2021-03-12 04:58:22.951 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] NVIDIA_REQUIRE_CUDA='cuda>=11.0 brand=tesla,driver>=418,driver<419 brand=tesla,driver>=440,driver<441 brand=tesla,driver>=450,driver<451'
2021-03-12 04:58:22.952 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] NVIDIA_VISIBLE_DEVICES='GPU-4d88dac3-b808-8c02-30cb-817671287688'
2021-03-12 04:58:22.953 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] OPTIND='1'
2021-03-12 04:58:22.955 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PATH='/home/.openmpi/bin:/opt/conda/bin:/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
2021-03-12 04:58:22.955 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PPID='1'
2021-03-12 04:58:22.956 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PS1='# '
2021-03-12 04:58:22.958 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PS2='> '
2021-03-12 04:58:22.958 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PS4='+ '
2021-03-12 04:58:22.960 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PWD='/'
2021-03-12 04:58:22.960 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PYTHONDONTWRITEBYTECODE='1'
2021-03-12 04:58:22.961 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PYTHONIOENCODING='UTF-8'
2021-03-12 04:58:22.962 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] PYTHONUNBUFFERED='1'
2021-03-12 04:58:22.963 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] SAGEMAKER_TRAINING_MODULE='sagemaker_pytorch_container.training:main'
2021-03-12 04:58:22.964 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] SHLVL='1'
2021-03-12 04:58:22.965 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] TORCH_CUDA_ARCH_LIST='3.5 3.7 5.2 6.0 6.1 7.0+PTX 8.0'
2021-03-12 04:58:22.966 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] TORCH_NVCC_FLAGS='-Xfatbin -compress-all'
2021-03-12 04:58:22.967 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] _='/bin/sh'
2021-03-12 04:58:22.968 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Setting up task environment.
2021-03-12 04:58:22.970 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Downloading code package.
2021-03-12 04:58:22.970 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Code package downloaded.
2021-03-12 05:02:12.929 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Bootstrapping environment.
2021-03-12 05:02:12.929 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Environment bootstrapped.
2021-03-12 05:02:18.122 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task is starting.
2021-03-12 05:02:18.122 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Fri Mar 12 05:02:15 2021
2021-03-12 05:02:18.122 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] +-----------------------------------------------------------------------------+
2021-03-12 05:02:18.123 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] | NVIDIA-SMI 450.51.06    Driver Version: 450.51.06    CUDA Version: 11.0     |
2021-03-12 05:02:18.123 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |-------------------------------+----------------------+----------------------+
2021-03-12 05:02:18.123 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
2021-03-12 05:02:18.124 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
2021-03-12 05:02:18.124 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |                               |                      |               MIG M. |
2021-03-12 05:02:18.124 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |===============================+======================+======================|
2021-03-12 05:02:18.125 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |   0  Tesla V100-SXM2...  Off  | 00000000:00:1E.0 Off |                    0 |
2021-03-12 05:02:18.125 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] | N/A   35C    P0    39W / 300W |      0MiB / 16160MiB |      0%      Default |
2021-03-12 05:02:18.125 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |                               |                      |                  N/A |
2021-03-12 05:02:18.125 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] +-------------------------------+----------------------+----------------------+
2021-03-12 05:02:18.126 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226]
2021-03-12 05:02:19.480 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] +-----------------------------------------------------------------------------+
2021-03-12 05:02:19.480 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] | Processes:                                                                  |
2021-03-12 05:02:19.480 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
2021-03-12 05:02:19.481 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |        ID   ID                                                   Usage      |
2021-03-12 05:02:19.481 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |=============================================================================|
2021-03-12 05:02:19.481 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] |  No running processes found                                                 |
2021-03-12 05:02:19.481 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] +-----------------------------------------------------------------------------+
2021-03-12 05:02:19.481 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] nvcc: NVIDIA (R) Cuda compiler driver
2021-03-12 05:02:19.481 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Copyright (c) 2005-2020 NVIDIA Corporation
2021-03-12 05:02:19.482 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Built on Wed_Jul_22_19:09:09_PDT_2020
2021-03-12 05:02:19.482 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Cuda compilation tools, release 11.0, V11.0.221
2021-03-12 05:02:19.482 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Build cuda_11.0_bu.TC445_37.28845127_0
2021-03-12 05:02:19.482 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] __Python VERSION: 3.6.11 | packaged by conda-forge | (default, Nov 27 2020, 18:57:37)
2021-03-12 05:02:19.482 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] [GCC 9.3.0]
2021-03-12 05:02:19.482 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] __pyTorch VERSION: 1.6.0
2021-03-12 05:02:19.482 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] __CUDA VERSION 10.2
2021-03-12 05:02:19.483 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] __CUDNN VERSION: 7605
2021-03-12 05:02:19.483 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] __Number CUDA Devices: 1
2021-03-12 05:02:21.931 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] __Devices
2021-03-12 05:02:21.931 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] index, name, driver_version, memory.total [MiB], memory.used [MiB], memory.free [MiB]
2021-03-12 05:02:21.932 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] 0, Tesla V100-SXM2-16GB, 450.51.06, 16160 MiB, 2 MiB, 16158 MiB
2021-03-12 05:02:21.932 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Active CUDA Device: GPU 0
2021-03-12 05:02:21.932 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Available devices  1
2021-03-12 05:02:21.932 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Current cuda device  0
2021-03-12 05:02:21.932 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] GPU count: 1
2021-03-12 05:02:21.932 [129/start/1341 (pid 5161)] [931bbcf6-5fa2-4661-83df-1d82d52b6226] Task finished with exit code 0.
2021-03-12 05:02:25.812 [129/start/1341 (pid 5161)] Task finished successfully.
2021-03-12 05:02:28.636 [129/end/1342 (pid 5193)] Task is starting.
2021-03-12 05:02:39.328 [129/end/1342 (pid 5193)] Task finished successfully.
2021-03-12 05:02:39.766 Done!
```

</details>

Some key information to check from the above output is
![img.png](/blog-theme/assets/img/posts/example-of-gpu-usage-in-metaflow/test_gpu_script_output.png)

## How does it work?

### NVIDIA Driver

The `NVIDIA-SMI` (represents the installation of NVIDIA driver) is obtained at EC2 instance level. Metaflow automatically picks an AMI with NVIDIA driver installed for your batch job when you request `resources(gpu=N)`. **Note that this WILL NOT happen if you don't have P2 or P3 instances in your Metaflow compute environment**

How to check P-series instances? Simply look at your compute environment
![img_1.png](/blog-theme/assets/img/posts/example-of-gpu-usage-in-metaflow/compute_env_instances.png)

### CUDA Toolkit

This should come with the image used. The example image above `763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.6.0-gpu-py36-cu110-ubuntu18.04` has CUDA installed.

#### Failed Example

If one wants to install CUDA through the `@conda` decorator in Metaflow, it sometimes **CAN FAIL**. To try, one can replace the decorator in the `test_gpu_script_output.py` with the following

```python
"""This will fail due to conflict version"""

@batch(
    cpu=2,
    gpu=1,
    memory=2400,
    image="763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.6.0-gpu-py36-cu110-ubuntu18.04"
)
@conda(libraries={
    'pytorch::pytorch': '1.6.0',  # need this conda decorator to install pytorch.
    # Metaflow will not automatically recognize the pytorch installed in the above image
    # because it will create its own conda env
    # The first pytorch before '::' is the pytorch conda channel
    # One wants to install from that channel for GPU support
    # The conda-forge channel's pytorch doesn't come with GPU support
    'anaconda::cudatoolkit': '11.0.221'
    # To find cudatoolkit version. Use `conda search cudatoolkit -c pytorch -c conda-forge -c anaconda
})
@step
def start(self):
    ...
```

Expected output should be

<details>
    <summary>Expected output (click to expand)</summary>

```shell
(base) root@2b15d8a4c149:/ds-model-mlm-metaflow/tests/test_metaflow_gpu# USERNAME=bz CONDA_CHANNELS=default,pytorch,conda-forge METAFLOW_PROFILE=ds-dev AWS_PROFILE=ds-dev-admin python test_metaflow_gpu.py --environment=conda run
Metaflow 2.2.7 executing TestGPUFlow for user:bz
Validating your flow...
    The graph looks good!
Running pylint...
    Pylint is happy!
Bootstrapping conda environment...(this could take a few minutes)
    Conda ran into an error while setting up environment.:
    Step: start, Error: UnsatisfiableError: The following specifications were found to be incompatible with each other:

    Output in format: Requested package -> Available versions

    Package libstdcxx-ng conflicts for:
    python==3.6.11 -> libstdcxx-ng[version='>=7.5.0|>=9.3.0']
    python==3.6.11 -> libffi[version='>=3.3,<3.4.0a0'] -> libstdcxx-ng[version='7.2.0.|>=4.9|>=7.3.0|>=7.2.0']

    Package cudatoolkit conflicts for:
    pytorch::pytorch==1.6.0 -> cudatoolkit[version='>=10.1,<10.2|>=10.2,<10.3|>=9.2,<9.3']
    anaconda|anaconda::cudatoolkit==11.0.221
    anaconda::cudatoolkit==11.0.221

    Package libgcc-ng conflicts for:
    python==3.6.11 -> libffi[version='>=3.3,<3.4.0a0'] -> libgcc-ng[version='7.2.0.|>=4.9|>=7.3.0|>=7.2.0']
    coverage==4.5.4 -> libgcc-ng[version='>=7.3.0']
    pytorch::pytorch==1.6.0 -> blas=[build=mkl] -> libgcc-ng[version='7.2.0.|>=4.9|>=7.2.0|>=7.3.0|>=7.5.0|>=9.3.0']
    anaconda::cudatoolkit==11.0.221 -> libgcc-ng[version='>=7.3.0']
    requests==2.24.0 -> python -> libgcc-ng[version='7.2.0.|>=4.9|>=7.3.0|>=7.5.0|>=9.3.0|>=7.2.0']
    boto3==1.14.47 -> python -> libgcc-ng[version='7.2.0.|>=4.9|>=7.3.0|>=7.5.0|>=9.3.0|>=7.2.0']
    python==3.6.11 -> libgcc-ng[version='>=7.5.0|>=9.3.0']
    click==7.1.2 -> python -> libgcc-ng[version='7.2.0.|>=4.9|>=7.3.0|>=7.5.0|>=9.3.0|>=7.2.0']
    coverage==4.5.4 -> python[version='>=3.8,<3.9.0a0'] -> libgcc-ng[version='7.2.0.|>=4.9|>=7.5.0|>=9.3.0|>=7.2.0']

    Package _libgcc_mutex conflicts for:
    anaconda::cudatoolkit==11.0.221 -> libgcc-ng[version='>=7.3.0'] -> _libgcc_mutex[version='|0.1',build='conda_forge|main']
    coverage==4.5.4 -> libgcc-ng[version='>=7.3.0'] -> _libgcc_mutex[version='|0.1',build='conda_forge|main']
    python==3.6.11 -> libgcc-ng[version='>=9.3.0'] -> _libgcc_mutex[version='|0.1',build='conda_forge|main']

    Package tzdata conflicts for:
    click==7.1.2 -> python -> tzdata
    boto3==1.14.47 -> python -> tzdata
    requests==2.24.0 -> python -> tzdata

    Package urllib3 conflicts for:
    boto3==1.14.47 -> botocore[version='>=1.17.47,<1.18.0'] -> urllib3[version='>=1.20,<1.26']
    requests==2.24.0 -> urllib3[version='>=1.21.1,<1.26,!=1.25.0,!=1.25.1']

    Package python conflicts for:
    click==7.1.2 -> python
    boto3==1.14.47 -> python
    pytorch::pytorch==1.6.0 -> python[version='>=3.6,<3.7.0a0|>=3.7,<3.8.0a0|>=3.8,<3.9.0a0']
    coverage==4.5.4 -> python[version='>=2.7,<2.8.0a0|>=3.6,<3.7.0a0|>=3.7,<3.8.0a0|>=3.8,<3.9.0a0']
    boto3==1.14.47 -> jmespath[version='>=0.7.1,<1.0.0'] -> python[version='2.7.|3.5.|3.6.|3.4.|>=3.5,<3.6.0a0|>=3.7,<3.8.0a0|>=2.7,<2.8.0a0|>=3.6,<3.7.0a0|>=3|>=3.8,<3.9.0a0|>=3.9,<3.10.0a0']
    requests==2.24.0 -> certifi[version='>=2017.4.17'] -> python[version='2.7.|3.5.|3.6.|>=2.7,<2.8.0a0|>=3.6,<3.7.0a0|>=3.7,<3.8.0a0|>=3.9,<3.10.0a0|>=3.8,<3.9.0a0|>=3.5,<3.6.0a0|<4.0']
    requests==2.24.0 -> python
    pytorch::pytorch==1.6.0 -> ninja -> python[version='2.7.|3.5.|3.6.|>=2.7,<2.8.0a0|>=3.5,<3.6.0a0|>=3.9,<3.10.0a0|3.4.']
    python==3.6.11

    Package python_abi conflicts for:
    pytorch::pytorch==1.6.0 -> ninja -> python_abi[version='3.6|3.6.|3.8.|3.7.|3.7|3.9.',build='_cp39|_cp37m|_pypy36_pp73|_cp36m|_cp38|_pypy37_pp73']
    click==7.1.2 -> python -> python_abi[version='3.6|3.7',build='_pypy36_pp73|_pypy37_pp73']
    boto3==1.14.47 -> python -> python_abi[version='3.6.|3.6|3.7|3.7.|3.8.|3.9.',build='_cp36m|_pypy36_pp73|_pypy37_pp73|_cp37m|_cp38|_cp39']
    requests==2.24.0 -> certifi[version='>=2017.4.17'] -> python_abi[version='2.7.|3.6.|3.6|3.7|3.7.|3.9.|3.8.',build='_cp36m|_pypy36_pp73|_cp37m|_pypy37_pp73|_cp39|_cp38|_cp27mu']
    coverage==4.5.4 -> python[version='>=3.7,<3.8.0a0'] -> python_abi[version='3.6|3.7',build='_pypy36_pp73|_pypy37_pp73']

    Package _openmp_mutex conflicts for:
    coverage==4.5.4 -> libgcc-ng[version='>=7.3.0'] -> _openmp_mutex[version='>=4.5']
    python==3.6.11 -> libgcc-ng[version='>=9.3.0'] -> _openmp_mutex[version='>=4.5']
    pytorch::pytorch==1.6.0 -> blas=[build=mkl] -> _openmp_mutex[version='|>=4.5',build=_llvm]
    anaconda::cudatoolkit==11.0.221 -> libgcc-ng[version='>=7.3.0'] -> _openmp_mutex[version='>=4.5']

    Package ca-certificates conflicts for:
    coverage==4.5.4 -> python[version='>=2.7,<2.8.0a0'] -> ca-certificates
    click==7.1.2 -> python -> ca-certificates
    boto3==1.14.47 -> python -> ca-certificates
    python==3.6.11 -> openssl[version='>=1.1.1h,<1.1.2a'] -> ca-certificates
    requests==2.24.0 -> python -> ca-certificates
```

</details>

## Summary

1. One uses pre-built images by
   [Amazon Deep Learning Images](https://github.com/aws/deep-learning-containers/blob/master/available_images.md)
   project. However these images are quite large (10GB+).
2. If one doesn't use an Amazon Deep Learning Image, he/she can leverage Metaflow(>=2.2.8)
   `@environment` decorator to set NVIDIA/CUDA environment variables to make GPU devices visible
   to the code that uses it. A simple example is
   ```python
    @environment(
        # environment decorator is used to set up global system level
        # environment variables
        vars={
            'NVIDIA_DRIVER_CAPABILITIES': 'compute,utility',  # necessary to allow computation on GPUs
            'CUDA_VISIBLE_DEVICES': '0,1',  # make two CUDA devices visible
        }
    )
   ```
3. Manually setting up the above environment variables **before importing torch** also works
   ```python
    @step
    def a_metaflow_step(self):
       import os
       # This trick is not only for Metaflow but for general Python scripts as well
       # Make CUDA devices visible before import torch
       os.environ['NVIDIA_DRIVER_CAPABILITIES'] = 'compute,utility'
       os.environ['CUDA_VISIBLE_DEVICES'] = '0,1'
       import torch
   ```
4. NVIDIA drivers should naturally comes with the P-series EC2 instances if you have a correctly managed Metaflow compute environment
5. Custom image, and manual installation of CUDA can be untrivial and results in version conflict.

### Related Issues from the Metaflow Community

1. [Github Issue #250](https://github.com/Netflix/metaflow/issues/250): Documentation / Explanation on how to use GPU
2. Two related conversations in Gitter:

   https://gitter.im/metaflow_org/community?at=604a5b9dd71b6554cd390ffa
   https://gitter.im/metaflow_org/community?at=6037f8d9cdbfc0620c2aedf2

pl-objectdetection_moc_ppc64
===============

- [pl-objectdetection_moc_ppc64](#pl-objectdetection-moc-ppc64)
  * [Description](#description)
  * [Usage](#usage)
    + [Requirements](#requirements)
    + [Docker run on x86_64](#docker-run-on-x86_64)
    + [Docker run on PowerPC](#docker-run-on-powerpc)
  * [Example](#example)
  * [Research and Development References](#research-and-development-references)
    + [Workflow](#workflow)
    + [Benchmarking result](#benchmarking-result)
  * [Troubleshoot](#troubleshoot)
      - [Error opening video stream or file](#error-opening-video-stream-or-file)
      - [Failed to establish a new connection](#failed-to-establish-a-new-connection)
  * [Related Links](#related-links)
    + [Docker Images](#docker-images)
  * [Contributor](#contributor)
  
Description
------------
```
                  ___  _     _           _     ____       _            _   _             
                 / _ \| |__ (_) ___  ___| |_  |  _ \  ___| |_ ___  ___| |_(_) ___  _ __  
                | | | | '_ \| |/ _ \/ __| __| | | | |/ _ \ __/ _ \/ __| __| |/ _ \| '_ \ 
                | |_| | |_) | |  __/ (__| |_  | |_| |  __/ ||  __/ (__| |_| | (_) | | | |
                 \___/|_.__// |\___|\___|\__| |____/ \___|\__\___|\___|\__|_|\___/|_| |_|
                          |__/                                                           


```

This is a GPU benchmarking plugin for ChRIS platform of Boston Children's Hospital ([What is ChRIS?](https://www.bu.edu/rhcollab/projects/radiology/))on both x86_64 and PowerPC MOC using object detection.

|Plugin Info  | Content | Description |
|:------:|:--------------:|:-----------------------------------------------:|
| Input | one video file | Target file to be tested with object detection |
| Output | one .csv file | Contains test results maximum frame per second, minimum frame per second and average frame per second  |



Usage
-------------------
### Requirements
Your host computer should be a linux os and installed CUDA 10.1 && nvidia container.

Main Dependencies:
`ffmpeg`
`opencv-python`
`tensorflow`
`tensorrt`

### Docker run on x86_64

First pull docker image to local environment:
```
docker pull docker.io/fnndsc/pl-objectdetection_x86
```
Then you can run it with parameters:
``` {.sourceCode .bash}
docker run --runtime=nvidia                                         \
            -e NVIDIA_VISIBLE_DEVICES=1                             \
            -v $(pwd)/in:/incoming -v $(pwd)/out:/outgoing          \
            docker.io/fnndsc/pl-objectdetection_x86                 \
            objectdetection.py                                      \
            -f animal360p.webm                                      \
            /incoming /outgoing
```
Parameters and meaning below in the table:

| **docker run parameters for x86_64**       |  |  |
|-----------------------------|-------------------------------------------------------------------------------------------------------------|------------------------------------------------|
| **parameters** | **function** | **example** |
| --runtime=nvidia | tells the docker to use the nvidia docker | --runtime=nvidia |
| -e | specifies the visible graphic device id | -e NVIDIA_VISIBLE_DEVICES=1 |
| -v | specify input and outgoing folder, check docker [volume bind](https://docs.docker.com/storage/bind-mounts/) | -v $(pwd)/in:/incoming -v $(pwd)/out:/outgoing |
| image_name | specify docker image name | docker.io/fnndsc/pl-objectdetection_x86 |
| script_file | specify script file to run | objectdetection.py |
| -f, --file | specify input file for object detection in input folder | -f animal360p.webm |

### Docker run on PowerPC
First pull docker image to local environment:
```
docker pull docker.io/fnndsc/docker.io/fnndsc/pl-objectdetection_moc_ppc64
```
Then you can run it with parameters:
``` {.sourceCode .bash}
docker run --security-opt label=type:nvidia_container_t     \
           -v $(pwd):/incoming:z -v $(pwd)/out:/outgoing:z  \
           docker.io/fnndsc/pl-matrixmultiply_moc_ppc64     \
           objectdetection.py                               \
           -f animal360p.webm                               \
           /incoming /outgoing
```
Parameters and meaning below in the table:


| **docker run parameters for PowerPC** |  |  |
|----------------------------------------------|-------------------------------------------------------------|-------------------------------------------------|
| **parameters** | **function** | **example** |
| --security-opt label=type:nvidia_container_t | tells the docker to use the nvidia docker | --security-opt label=type:nvidia_container_t |
| -v | specify input and outgoing folder, check docker [volume bind](https://docs.docker.com/storage/bind-mounts/) | -v $(pwd):/incoming:z -v $(pwd)/out:/outgoing:z |
| image_name | specify docker image name | docker.io/fnndsc/pl-objectdetection_x86 |
| script_file | specify script file to run | objectdetection.py |
| -f, --file | specify input file for object detection in input folder | -f animal360p.webm |

**Build Instructions**

For ppc64le image, we cannot use the automatic build on docker hub. We have to build this conatiner locally and push it into docker hub.

```bash
cd path/to/this/repo
docker login docker.io -u [your docker.io username]
docker build -f Dockerfile -t docker.io/fnndsc/pl-objectdetection_moc_ppc64 .
docker push "docker.io/fnndsc/pl-objectdetection_moc_ppc64"

```
Example
-------
For both `x86_64` and `PowerPC`, please check `FNNDSC/objectdetection_example` repo for example usage:[objectdetection_example](https://github.com/FNNDSC/objectdetection_example)

Research and Development References
-------------

### Workflow

![workflow](https://miro.medium.com/max/1400/0*mnywPWQIQW5j0Paf)


This graph show the workflow of the original python script (provided by nVidia). One of the difference in our scripts is that we use a file instead of the web camera as graph source. Therefore, this container have to use the ffmpeg to decode the video file. Also, since there is no graphic interface in the server/moc/openshift, we removed the realtime progress showing codes and replace it by saving the output to an output file (output.avi) in the `outgoing` directiory. This means ffmpeg is essentical.

There is another output file called `FramePerSecondRecord.csv`. This file contains the benchmarking results of the plugin. The output should be like this:
| maximum_fps  | minimum_fps | average_fps |
|:------------:|:-----------:|:-----------:|
|                                                                                                                            250.0                                                                                                                          |                                                                     142.86                                                                    |                                                                                                                      239.92                                                                                                                    |

If you wanna more research details of this project, [check this tutorial.](https://medium.com/better-programming/real-time-object-detection-on-gpus-in-10-minutes-6e8c9b857bb3)


(If you run it multiple times , the newest result will be added to the last line of file.)

(Results from a `ppc64le` machine)

This shows the information about the inference time for every frame. We think it shows the data bus latency from cpu/main memory to the GPU.

### Benchmarking result
On `ppc64le` machine, the typical inference time for each frame is about 4 ms. However in `x86_64` machine, we got about 6~7 ms inference time for every frame. We think the differnece is significant (powerpc is about 40% faster than `x86_64`).


Troubleshoot
-------

#### Error opening video stream or file

This means the opencv didn't open the video file successfully. Check:

* If the file exist
* If the input video coding format is supported by curent version ffmpeg.

#### Failed to establish a new connection

Please contact the machine administrator to ensure the docker has the internet access ability.

Related Links
----------
Most python scripts in this repo is forked from this tensorRT example provided by Nvidia:

https://github.com/NVIDIA/object-detection-tensorrt-example

### Docker Images
PowerPC: https://hub.docker.com/r/fnndsc/pl-objectdetection_moc_ppc64

x86_64: https://hub.docker.com/repository/docker/fnndsc/pl-objectdetection_moc_ppc64



Contributor
---
Haoyang Wang  <haoyangw@bu.edu>

Kefan Zhang <kefan29@bu.edu>

Feel free to reach out us if you have any question. :)

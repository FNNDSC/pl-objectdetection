pl-objectdetection
==================

An benchmarking plugin using object detection for Chris platform.
```

  ___  _     _           _     ____       _            _   _             
 / _ \| |__ (_) ___  ___| |_  |  _ \  ___| |_ ___  ___| |_(_) ___  _ __  
| | | | '_ \| |/ _ \/ __| __| | | | |/ _ \ __/ _ \/ __| __| |/ _ \| '_ \ 
| |_| | |_) | |  __/ (__| |_  | |_| |  __/ ||  __/ (__| |_| | (_) | | | |
 \___/|_.__// |\___|\___|\__| |____/ \___|\__\___|\___|\__|_|\___/|_| |_|
          |__/                                                           



```

Contributer
---
Haoyang Wang  haoyangw@bu.edu

Kefan   Zhang kefan29@bu.edu

Feel free to reach out us if you have any question. :)

Abstract
--------
Most python scripts in this repo is forked from this tensorRT example provided by Nvidia:

https://github.com/NVIDIA/object-detection-tensorrt-example

and it's modified to use a file as the input, run on a cloud machine and become a chris plugin.

Docker image
------

docker.io/fnndsc/pl-objectdetection_moc_ppc64

https://hub.docker.com/repository/docker/fnndsc/pl-objectdetection_moc_ppc64

### Build instructions

For ppc64le image, we cannot use the automatic build on docker hub. We have to build this conatiner locally and push it into docker hub.

```bash
    cd path/to/this/repo
    docker login docker.io -u [your docker.io username]
    docker build -f Dockerfile -t docker.io/fnndsc/pl-objectdetection_moc_ppc64 .
    docker push "docker.io/fnndsc/pl-objectdetection_moc_ppc64"

```

## Deploy requirement
Your host computer should be a linux os and installed CUDA 10.1 && nvidia container.

For ppc64le machine:

https://github.com/FNNDSC/pl-objectdetection_moc_ppc64

For x86_64 machine:

https://github.com/FNNDSC/pl-objectdetection_x86

### main Dependencies
ffmpeg
opencv-python
tensorflow
tensorrt

Run
---

### Using `docker run`

Check this repo for more information https://github.com/FNNDSC/objectdetection_example

To run using `docker`, be sure to assign an "input" directory to
`/incoming` and an output directory to `/outgoing`. *Make sure that the*
`$(pwd)/out` *directory is world writable!*
And use the `--f` flag to set the inputfile inside your `/incoming` directory.

Now, prefix all calls with

``` {.sourceCode .bash}
docker run --security-opt label=type:nvidia_container_t    \
           -v $(pwd):/incoming:z -v $(pwd)/out:/outgoing:z \
           docker.io/fnndsc/pl-matrixmultiply_moc_ppc64    \
           objectdetection.py                               \
           -f animal360p.webm /incoming /outgoing
```


How it works
------

*Get detailed information from: https://medium.com/better-programming/real-time-object-detection-on-gpus-in-10-minutes-6e8c9b857bb3*

### Workflow

*this image powered by previous url*

![workflow](https://miro.medium.com/max/1400/0*mnywPWQIQW5j0Paf)

This graph show the workflow of the original python script (provided by nVidia). One of the difference in our scripts is that we use a file instead of the web camera as graph source. Therefore, this container have to use the ffmpeg to decode the video file. Also, since there is no graphic interface in the server/moc/openshift, we removed the realtime progress showing codes and replace it by saving the output to an output file (output.avi) in the `outgoing` directiory. This means ffmpeg is essentical.

There is another output file called `FramePerSecondRecord.csv`. This file contains the benchmarking results of the plugin. The output should be like this:
```
maximum_fps,minimum_fps,average_fps
250.0,142.86,239.92
```
*(If you run it multiple times , the newest result will be added to the last line of file.)*

*(Result is gotten from a ppc64le machine)*

This shows the information about the inference time for every frame. We think it shows the data bus latency from cpu/main memory to the GPU.
### Benchmarking result.
On ppc64le machine, the typical inference time for each frame is about 4 ms. However in x86_64 machine, we got about 6~7 ms inference time for every frame. We think the differnece is significant (powerpc is about 40% faster than x86_64).
### Troubleshoot

#### Error opening video stream or file

This means the opencv didn't open the video file successfully. Check:

* If the file exist
* If the input video coding format is supported by curent version ffmpeg.

#### Failed to establish a new connection

Please contact the machine administrator to ensure the docker has the internet access ability.

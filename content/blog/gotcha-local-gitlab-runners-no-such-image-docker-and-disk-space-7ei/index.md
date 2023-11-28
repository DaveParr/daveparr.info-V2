+++
title = "Local gitlab runners, 'no such image', docker and disk space"
description = "A quick note on how to fix a gitlab runner error when running locally"
date = 2020-01-10
[taxonomies]
tags = ["cicd", "docker", "gitlab", "gotchas"]
+++

{% crt() %}

```
Davids-MacBook-Pro:data-science davidparr$ gitlab-runner exec docker 'anomaly detection'
Runtime platform                                    arch=amd64 os=darwin pid=72331 revision=a8a019e0 version=12.3.0
WARNING: You most probably have uncommitted changes. 
WARNING: These changes will not be tested.         
Running with gitlab-runner 12.3.0 (a8a019e0)
Using Docker executor with image rocker/tidyverse:latest ...
Authenticating with credentials from /Users/davidparr/.docker/config.json
Pulling docker image rocker/tidyverse:latest ...
ERROR: Preparation failed: Error: No such image: rocker/tidyverse:latest (executor_docker.go:195:0s)
Will be retried in 3s ...
Using Docker executor with image rocker/tidyverse:latest ...
Authenticating with credentials from /Users/davidparr/.docker/config.json
Pulling docker image rocker/tidyverse:latest ...
ERROR: Preparation failed: Error: No such image: rocker/tidyverse:latest (executor_docker.go:195:0s)
Will be retried in 3s ...
Using Docker executor with image rocker/tidyverse:latest ...
Authenticating with credentials from /Users/davidparr/.docker/config.json
Pulling docker image rocker/tidyverse:latest ...
ERROR: Preparation failed: Error: No such image: rocker/tidyverse:latest (executor_docker.go:195:0s)
Will be retried in 3s ...
ERROR: Job failed (system failure): Error: No such image: rocker/tidyverse:latest (executor_docker.go:195:0s)
```
{% end %}
But you _know_ the image exists. It's _definately_ a thing. Look, [it's right here](https://hub.docker.com/r/rocker/tidyverse/). I can even __RUN IT IN PRODUCTION__ so why is it not running on my machine?

![A shrugging emogi with the text: it works in production](./4t9ogf06iqphsaewwxvt.jpg)

Turns out gitlab, as brilliant as I'm normally finding their CICD solution, is lying to you. The image _does_ exist, you _aren't_ mad, just not seeing the whole picture. 

{% crt() %}

```
Davids-MacBook-Pro:data-science davidparr$ docker pull rocker/tidyverse
Using default tag: latest
latest: Pulling from rocker/tidyverse
16ea0e8c8879: Pull complete 
7ce39da2c1e2: Extracting [==================================================\u003e]  222.8MB/222.8MB
ff1bceed0bef: Download complete 
e36d273bec5a: Download complete 
d3acc34c6c77: Download complete 
14d07989ce8b: Download complete 
73b6bcbfcb26: Download complete 
70b803ec0e47: Download complete 
failed to register layer: Error processing tar file(exit status 1): write /usr/lib/gcc/x86_64-linux-gnu/8/libsupc++.a: no space left on device
Davids-MacBook-Pro:data-science davidparr$ docker pull rocker/tidyverse
Using default tag: latest
latest: Pulling from rocker/tidyverse
16ea0e8c8879: Pull complete 
7ce39da2c1e2: Extracting [===================\u003e                               ]  85.23MB/222.8MB
ff1bceed0bef: Download complete 
e36d273bec5a: Download complete 
d3acc34c6c77: Download complete 
14d07989ce8b: Download complete 
73b6bcbfcb26: Download complete 
70b803ec0e47: Downloading [==================================================\u003e]  324.8MB/324.8MB
write /var/lib/docker/tmp/GetImageBlob686008714: no space left on device
```
{% end %}

_That's_ the error message we needed. It turns out that after a while, your local machines 'Sparse Image', that has all your docker images, containers, networks and registries, and also resizes it's on disk footprint as you use docker, will get filled up with old images, containers and other cruft. 

There is a bit more info [in this thread](https://forums.docker.com/t/no-space-left-on-device-error/10894) but the short version is that you probably just want to run `docker system prune -a`.

This command is documented to \"Remove unused data\", and when run will tell you all about what it's going to do:

```
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache
```

For my use cases this is perfect. All the images I need in practice are backed up on the cloud and can be rebuilt from code when needed (as they should be), leaving my disk memory to be able to be treated a little more like active memory, hot swapping in and out containers as needed, keeping the disk space tidy and minimal. 

In the end I received this lovely message:
```
Total reclaimed space: 10.24GB
```

And we're back to happy local emulation of my CI/CD pipeline. Now if only I could optimise my R testing image...

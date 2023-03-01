## Chapter 4
Packaging applications
from source code into
Docker Images


It would be much cleaner to package the build toolset once and share it, which is
exactly what you can do with Docker. You can write a Dockerfile that scripts the
deployment of all your tools, and build that into an image. Then you can use that
image in your application Dockerfiles to compile the source code, and the final out-
put is your packaged application.

To understand this see the below MultiStage Docker File.

```docker
FROM diamol/base AS build-stage
RUN echo 'Building...' > /build.txt
FROM diamol/base AS test-stage
COPY --from=build-stage /build.txt /build.txt
RUN echo 'Testing...' >> /build.txt
FROM diamol/base
COPY --from=test-stage /build.txt /build.txt
CMD cat /build.txt
```
This is called a multi-stage Dockerfile, because there are several stages to the build.
Each stage starts with a FROM instruction, and you can optionally give stages a name
with the AS parameter.

The docker file has three stages: **build-stage** , **test-stage** , and the
**final** unnamed stage. Although there are multiple stages, the output will be a single
Docker image with the contents of the final stage.
-   Each stage runs independently, but you can copy files and directories from previous
stages. I’m using the COPY instruction with the --from argument, which tells Docker to
copy files from an earlier stage in the Dockerfile, rather than from the filesystem of the
host computer. In this example I generate a file in the build stage, copy it into the test
stage, and then copy the file from the test stage into the final stage.
-   There’s one new instruction here, RUN , which I’m using to write files. The RUN
instruction executes a command inside a container during the build, and any out-
put from that command is saved in the image layer. You can execute anything in a
RUN instruction, but the commands you want to run need to exist in the Docker
image you’re using in the FROM instruction.
-   It’s important to understand that the individual stages are isolated. You can use differ-
ent base images with different sets of tools installed and run whatever commands you
like. The output in the final stage will only contain what you explicitly copy from ear-
lier stages. If a command fails in any stage, the whole build fails.


This approach makes your application truly portable. You can run the app in a con-tainer  anywhere,  but  you  can  also  build  the  app  anywhere—Docker  is  the  only  pre-requisite. Your build server just needs Docker installed; new team members get set upin  minutes,  and  the  build  tools  are  all  centralized  in  Docker  images,  so  there’s  nochance of getting out of sync.

Containers access each other across a virtual network, using the virtual IPaddress that Docker allocates when it creates the container. You can create and man-age virtual Docker networks through the command line.

-   Create  a  Docker  network  for  containers  to  communicate  witheach other:
```bash
docker network create nat
```


-   you can explicitly connect them to that Docker network using the **--network** flag, and any containers on that network can reach each other using the container names.
-   Run a container from the image, publishing port 80 to the hostcomputer, and connecting to the nat network:
```bash
docker container run --name iotd -d -p 800:80 --network nat image-of-the-day
```


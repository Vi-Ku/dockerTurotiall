## Notes of docker book

----
----
-----

### **Chapter 2 Notes**
---
Ff you want clean up all the containers at the
end of every chapter by running this command:
```bash
docker container rm -f $(docker container ls -aq)
```

And if you want to reclaim disk space after following the exercises, you can run this
command:
```bash
docker image rm -f $(docker image ls -f reference='diamol/*' -q)
```
`
Docker is smart about downloading what it needs, so you can safely run these com-
mands at any time. The next time you run containers, if Docker doesn’t find what it
needs on your machine, it will download it for you.
`



-   To start a interactive bash shell session in the container from the terminal
```bash
docker container run --interactive --tty diamol/base
```
`The --interactive flag tells Docker you want to set up a connection to the container,
and the --tty flag means you want to connect to a terminal session inside the container.
`
-   Run the commands *hostname* and *date* and you’ll see details of
the container’s environment:
-   Open up a new terminal session, and you can get details of all
the running containers with this command:
```bash
docker container ls
```
-   docker container top lists the processes running in the con-
tainer. I’m using f1 as a short form of the container ID f1695de1f2ec :
```bash
docker container top f1
```
-   docker container logs displays any log entries the container
has collected:
```bash
docker container logs f1
```
-  docker container inspect shows you all the details of a
container:
```bash
docker container inspect f1
```
`
Docker adds a consistent manage-
ment layer on top of every application. You could have a 10-year-old Java app running
in a Linux container, a 15-year-old .NET app running in a Windows container, and a
brand-new Go application running on a Raspberry Pi. You’ll use the exact same com-
mands to manage them— run to start the app, logs to read out the logs, top to see the
processes, and inspect to get the details.
`
-   **docker container
ls**   will show that you have no containers, because the command only shows running containers.
-   Run docker container ls --all , which shows all containers in
any status:
```bash
docker container ls --all
```
`First, containers are running only while the application inside the container is run-
ning. As soon as the application process ends, the container goes into the exited state.
Exited containers don’t use any CPU time or memory.
econd, containers don’t disappear when they exit. Containers in the exited state
still exist, which means you can start them again, check the logs, and copy files to and
from the container’s filesystem. You only see running containers with docker container
ls , but Docker doesn’t remove exited containers unless you explicitly tell it to do so.
Exited containers still take up space on disk because their filesystem is kept on the
computer’s disk.
`
-   So what about starting containers that stay in the background and just keep run-
ning? That’s actually the main use case for Docker: running server applications like
websites, batch processes, and databases.
```bash
docker container run --detach --publish 8088:80 diamol/ch02-hello-
diamol-web
```
-   Containers that sit in the background and listen for network traffic (HTTP requests in this case) need a couple
of extra flags in the container run command:

    1.  --detach —Starts the container in the background and shows the container ID
    2.  --publish —Publishes a port from the container to the computer

`
Running a detached container just puts the container in the background so it starts
up and stays hidden, like a Linux daemon or a Windows service. Publishing ports
needs a little more explanation. When you install Docker, it injects itself into your
computer’s networking layer. Traffic coming into your computer can be intercepted
by Docker, and then Docker can send that traffic into a container.
`
`
Containers aren’t exposed to the outside world by default. Each has its own IP
address, but that’s an IP address that Docker creates for a network that Docker
manages—the container is not attached to the physical network of the computer.
Publishing a container port means Docker listens for network traffic on the computer
port, and then sends it into the container. In the preceding example, traffic sent to
the computer on port 8088 will get sent into the container on port 80 as can be seen in the below figure
`


![/images/docker-1.png](docker)

-  **docker container stats** is another useful one: it shows a live
view of how much CPU, memory, network, and disk the container is using.
The output is slightly different for Linux and Windows containers:
```bash
docker container stats e53
```
-   When you’re done working with a container, you can remove it with **docker container
rm** and the container ID, using the **--force** flag to force removal if the container is
still running.

-   Run this command to remove all your containers:
```bash
docker container rm --force $(docker container ls --all --quiet)
```

-   what’s actually happening when you run applications with Docker. Installing
Docker and running containers is deceptively simple—there are actually a few differ-
ent components involved
    1.  The Docker Engine is the management component of Docker. It looks after the
local image cache, downloading images when you need them, and reusing them if they’re already downloaded. It also works with the operating system to
create containers, virtual networks, and all the other Docker resources. The
Engine is a background process that is always running (like a Linux daemon or
a Windows service).
    2.  The Docker Engine makes all the features available through the Docker API,
which is just a standard HTTP-based REST API. You can configure the Engine
to make the API accessible only from the local computer (which is the default),
or make it available to other computers on your network.
    3.  The Docker command-line interface (CLI) is a client of the Docker API. When you
run Docker commands, the CLI actually sends them to the Docker API, and the
Docker Engine does the work.




------


# Chapter 2 Notes

`Here’s some interesting output from the docker image pull command, which
shows you how images are stored. A Docker image is logically one thing—you can
think of it as a big zip file that contains the whole application stack. This image has the
Node.js runtime together with my application code.
During the pull you don’t see one single file downloaded; you see lots of down-
loads in progress. Those are called image layers. A Docker image is physically
stored as lots of small files, and Docker assembles them together to create the con-
tainer’s filesystem. When all the layers have been pulled, the full image is available
to use.
`

-   Let’s run a container from the image and see what the app does:
```bash
docker container run -d --name web-ping diamol/ch03-web-ping
```
The **-d** flag is a short form of **--detach** , so this container will run in the background.
The application runs like a batch job with no user interface. Unlike the website con-
tainer we ran detached in chapter 2, this one doesn’t accept incoming traffic, so you
don’t need to publish any ports.

There’s one new flag in this command, which is **--name** . You know that you can
work with containers using the ID that Docker generates, but you can also give them a
friendly name. This container is called web-ping , and you can use that name to refer
to the container instead of using the random ID.


-   Environment variables are just key/value pairs that the operating system provides.
They work in the same way on Windows and Linux, and they’re a very simple way to store small pieces of data. Docker containers also have environment variables, but
instead of coming from the computer’s operating system, they’re set up by Docker in
the same way that Docker creates a hostname and IP address for the container.
-   Remove the existing container, and run a new one with a value
specified for the TARGET environment variable:
```bash
docker rm -f web-ping
docker container run --env TARGET=google.com diamol/ch03-web-ping
```

This container is doing something different. First, it’s running interactively because
you didn’t use the **--detach** flag, so the output from the app is shown on your con-
sole. The container will keep running until you end the app by pressing **Ctrl-C**. Second,
it’s pinging google.com now instead of blog.sixeyed.com.


### Writing your first Dockerfile
Example 1
```dockerfile
FROM diamol/node
ENV TARGET="blog.sixeyed.com"
ENV METHOD="HEAD"
ENV INTERVAL="3000"
WORKDIR /web-ping
COPY app.js .
CMD ["node", "/web-ping/app.js"]
```
Here’s the breakdown for each instruction used in the example:
-   FROM — Every image has to start from another image. In this case, the web-ping
image will use the diamol/node image as its starting point. That image has
Node.js installed, which is everything the web-ping application needs to run.
-   ENV —Sets values for environment variables. The syntax is [key]="[value]" ,
and there are three ENV instructions here, setting up three different environ-
ment variables.
-   WORKDIR —Creates a directory in the container image filesystem, and sets that to
be the current working directory. The forward-slash syntax works for Linux and
Windows containers, so this will create /web-ping on Linux and C:\web-ping
on Windows.
-   COPY —Copies files or directories from the local filesystem into the container
image. The syntax is [source path] [target path] —in this case, I’m copying
app.js from my local machine into the working directory in the image.
-   CMD —Specifies the command to run when Docker starts a container from the
image. This runs Node.js, starting the application code in app.js.

Turn this Dockerfile into a Docker image by running docker
image build :
```bash
docker image build --tag web-ping .
```
-   The **--tag** argument is the name for the image, and the final argument is the direc-
tory where the Dockerfile and related files are. Docker calls this directory the “con-
text,” and the period means “use the current directory.” You’ll see output from the
build command, executing all the instructions in the Dockerfile.


### Understanding Docker images and image layers
The Docker image contains all the files you packaged, which become the con-
tainer’s filesystem, and it also contains a lot of metadata about the image itself. That
includes a brief history of how the image was built. You can use that to see each layer
of the image and the command that built the layer.
-   Check the history for your web-ping image:
```bash
docker image history web-ping
```
-   The size column you see is the logical size of the image—that’s
how much disk space the image would use if you didn’t have any other images on your
system. If you do have other images that share layers, the disk space Docker uses is
much smaller. You can’t see that from the image list, but there are Docker system com-
mands that tell you more.
-   The system df command shows exactly how much disk
space Docker is using:
```bash
docker system df
```
-   If image layers are shared around, they can’t be edited—
otherwise a change in one image would cascade to all the other images that share the
changed layer. Docker enforces that by making image layers read-only. Once you cre-
ate a layer by building an image, that layer can be shared by other images, but it can’t
be changed. You can take advantage of that to make your Docker images smaller and
your builds faster by optimizing your Dockerfiles.


`
Any Dockerfile you write should be optimized so that the instructions are ordered
by how frequently they change—with instructions that are unlikely to change at the
start of the Dockerfile, and instructions most likely to change at the end. The goal is for most builds to only need to execute the last instruction, using the cache for
everything else. That saves time, disk space, and network bandwidth when you start
sharing your images.
`

----

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


----
# Chapter 5:
## Sharing images with Docker Hub and other registries.

Sharing is all about tak-ing the images you’ve built on your local machine and making them available forother  people  to  use. 

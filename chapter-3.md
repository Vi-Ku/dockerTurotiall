

# Chapter 3 Notes

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

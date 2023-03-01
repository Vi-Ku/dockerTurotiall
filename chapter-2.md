
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



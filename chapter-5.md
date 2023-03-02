## Sharing images with Docker Hub and other registries.

Sharing is all about tak-ing the images you’ve built on your local machine and making them available forother  people  to  use. 

Open a terminal session and save your Docker Hub ID in a vari￾able. Your Docker ID is your username, not your email address. This is one
command that is different on Windows and Linux, so you’ll need to choose
the right option:
```powershell
# using PowerShell on Windows
$dockerId="<your-docker-id-goes-here>"
```

```bash
# using Bash on Linux or Mac
export dockerId="<your-docker-id-goes-here>"
```

Log in to Docker Hub. Hub is the default registry, so you don’t
need to specify a domain name:
```bash
docker login --username $dockerId
```

Create a new reference for your existing image, tagging it as ver￾sion 1:
```bash
docker image tag image-gallery $dockerId/image-gallery:v1
```

List the image-gallery image references:
```bash
docker image ls --filter reference=image-gallery --filter 
reference='*/image-gallery'
```

The docker
image push command is the counterpart of the pull command; it uploads your local
image layers to the registry.

List the image-gallery image references:
```bash
docker image push $dockerId/image-gallery:v1
```

The fact that registries work with image layers is another reason why you need to
spend time optimizing your Dockerfiles. Layers are only physically uploaded to the
registry if there isn’t an existing match for that layer’s hash. It’s like your local Docker
Engine cache, but applied across all images on the registry. If you optimize to the
point where 90% of layers come from the cache when you build, 90% of those layers
will already be in the registry when you push. Optimized Dockerfiles reduce build
time, disk space, and network bandwidth.
 You can browse to Docker Hub now and check your image. The Docker Hub UI
uses the same repository name format as image references, so you can work out the
URL of your image from your account name.


This little script writes the URL to your image’s page on Docker
Hub:
```bash
echo "https://hub.docker.com/r/$dockerId/image-gallery/tags"
```

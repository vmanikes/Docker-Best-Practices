# Docker Image Best Practices

## Use Minimal Base Images
Using a minimal base image rather than a full blown OS (Unless you require any OS libraries) image in `FROM` in your dockerfile helps in drastically reducing your container image size. By preferring minimal images that bundle only the necessary system tools and libraries required to run your project, you are also minimizing the attack surface for attackers and ensuring that you ship a secure OS

## Sign and verify images
- Authenticity of Docker images is a challenge. We need to make sure that when we pull the image from registry, the image is not altered in any way by a third party. Make it a best practice to `always verify images before pulling them`.

- To experiment with verification, temporarily enable Docker Content Trust with the following command:
```
export DOCKER_CONTENT_TRUST=1
```
- When you now try to pull the image that is not signed, `the request is denied and image is not pulled`
- Always perefer the official docker certified images rather than a 3rd party. To sign the images use [Docker Notary](https://docs.docker.com/notary/getting_started/). It will verify the signature for you and blocks running an image whose signature is invalid.
For more detailed instructions, refer to [Docs](https://docs.docker.com/engine/security/trust/content_trust/)

## Avoiding sensitive information to docker images
- Sometimes images need secrets such as an SSH private key to pull code from a private repository, or you need tokens to install private packages. If you copy them into the Docker intermediate container they are cached on the layer to which they were added, even if you delete them later on. These tokens and keys must be kept outside of the `Dockerfile`
- **Avoid Recursive Copy**: The command `COPY . .` copies the entire build context to the container which contains sensitive files. If you have sensitive files in your build context folder user `.dockerignore`

## Use COPY instead of ADD
## Use multi stage builds
- When building a docker image, many  artifacts are created that are only needed for build time such as temp dependencies for testing etc.
- Keeping these artifacts in base image can result in increased base image size and can also increase the attack surface.
- Golang is a great example. To build a Golang application, you need the Go compiler. The compiler produces an executable that runs on any operating system, without dependencies, including scratch images
- Assume this simple go program
```
package main

import "fmt"

func main() {
	fmt.Println("Hello world!")
}
```
If I use the `single-stage` build with this Dockerfile, The image size would be around `258 MB`
```
FROM golang:alpine
WORKDIR /app
ADD . /app
RUN cd /app && go build -o goapp
ENTRYPOINT ./goapp
```
Let's try the multi stage build
```
# build stage
FROM golang:alpine AS build-env
ADD . /src
RUN cd /src && go build -o goapp

# final stage
FROM alpine
WORKDIR /app
COPY --from=build-env /src/goapp /app/
ENTRYPOINT ./goapp
```

When you inspect the image size with multi-sttage build, the image size is around `6 MB`

## Use a Linter
- Adopt the use of a linter to avoid common mistakes and establish best practice guidelines that engineers can follow in an automated way. One such linter is [hadolint](https://github.com/hadolint/hadolint). It parses a Dockerfile and shows a warning for any errors that do not match its best practice rules
- Use it with VS Code to write Dockerfiles faster



# Credits
[snyk.io](https://snyk.io/blog/10-docker-image-security-best-practices/)
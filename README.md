# docker-slim: Lean and Mean Docker containers

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [docker-slim: Lean and Mean Docker containers](#docker-slim-lean-and-mean-docker-containers)
  - [DESCRIPTION](#description)
  - [NEW](#new)
  - [INSTALLATION](#installation)
  - [BASIC USAGE INFO](#basic-usage-info)
  - [QUICK SECCOMP EXAMPLE](#quick-seccomp-example)
  - [USING AUTO-GENERATED SECCOMP PROFILES](#using-auto-generated-seccomp-profiles)
  - [ORIGINAL DEMO VIDEO](#original-demo-video)
  - [DEMO STEPS](#demo-steps)
  - [USAGE DETAILS](#usage-details)
  - [DOCKER CONNECT OPTIONS](#docker-connect-options)
  - [HTTP PROBE COMMANDS](#http-probe-commands)
  - [MINIFYING COMMAND LINE TOOLS](#minifying-command-line-tools)
  - [CURRENT STATE](#current-state)
  - [FAQ](#faq)
    - [Is it safe for production use?](#is-it-safe-for-production-use)
    - [How can I contribute if I don't know Go?](#how-can-i-contribute-if-i-dont-know-go)
    - [What's the best application for DockerSlim?](#whats-the-best-application-for-dockerslim)
    - [Can I use DockerSlim with dockerized command line tools?](#can-i-use-dockerslim-with-dockerized-command-line-tools)
  - [BUILD PROCESS](#build-process)
      - [Local Build Steps](#local-build-steps)
      - [Traditional Go Way to Build](#traditional-go-way-to-build)
      - [Builder Image Steps](#builder-image-steps)
  - [DESIGN](#design)
    - [CORE CONCEPTS](#core-concepts)
    - [DYNAMIC ANALYSIS OPTIONS](#dynamic-analysis-options)
    - [SECURITY](#security)
    - [CHALLENGES](#challenges)
  - [DEVELOPMENT PROGRESS](#development-progress)
    - [PHASE 3 (WIP)](#phase-3-wip)
    - [TODO](#todo)
  - [ORIGINS](#origins)
  - [ONLINE](#online)
  - [MINIFIED DOCKER HUB IMAGES](#minified-docker-hub-images)
  - [NOTES](#notes)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## DESCRIPTION


Creating small containers requires a lot of voodoo magic and it can be pretty painful. You shouldn't have to throw away your tools and your workflow to have skinny containers. Using Docker should be easy.

`docker-slim` is a magic diet pill for your containers :) It will use static and dynamic analysis to create a skinny container for your app.

## NEW

Latest version: 1.14 (3/13/2016)

* Auto-generated seccomp profiles for Docker 1.10.
* Python 3 support
* Docker connect options
* HTTP probe commands
* Include extra directories and files in minified images

## INSTALLATION

1. Download the zip package for your platform.
   - [Latest Mac binaries](https://github.com/cloudimmunity/docker-slim/releases/download/1.14/dist_mac.zip)
   - [Latest Linux binaries](https://github.com/cloudimmunity/docker-slim/releases/download/1.14/dist_linux.zip)
2. Unzip the package.
3. Add the location where you unzipped the package to your PATH environment variable (optional).

If the directory where you extracted the binaries is not in your PATH then you'll need to run your `docker-slim` commands from that directory.

## BASIC USAGE INFO

`docker-slim [info|build|profile] [--http-probe|--remove-file-artifacts] <IMAGE_ID_OR_NAME>`

Example: `docker-slim build --http-probe my/sample-node-app`

To generate a Dockerfile for your "fat" image without creating a new "slim" image use the `info` command.

Example: `docker-slim info 6f74095b68c9`

## QUICK SECCOMP EXAMPLE

If you want to auto-generate a Seccomp profile AND minify your image use the `build` command. If you only want to auto-generate a Seccomp profile (along with other interesting image metadata) use the `profile` command.

Step one: run DockerSlim

`docker-slim build --http-probe your-name/your-app`

Step two: use the generated Seccomp profile

`docker run --security-opt seccomp:<docker-slim directory>/.images/<YOUR_APP_IMAGE_ID>/artifacts/your-name-your-app-seccomp.json <your other run params> your-name/your-app`

Feel free to copy the generated profile :-)

You can use the generated Seccomp profile with your original image or with the minified image.

## USING AUTO-GENERATED SECCOMP PROFILES

You can use the generated profile with your original image or with the minified image DockerSlim created:

`docker run --security-opt seccomp:path_to/my-sample-node-app-seccomp.json -p 8000:8000 my/sample-node-app.slim`

## ORIGINAL DEMO VIDEO

[![DockerSlim demo](http://img.youtube.com/vi/uKdHnfEbc-E/0.jpg)](https://www.youtube.com/watch?v=uKdHnfEbc-E)

[Demo video on YouTube](https://youtu.be/uKdHnfEbc-E)

## DEMO STEPS

The demo run on Mac OS X, but you can build a linux version. Note that these steps are different from the steps in the demo video.

0. Get the docker-slim [Mac](https://github.com/cloudimmunity/docker-slim/releases/download/1.14/dist_mac.zip) or [Linux](https://github.com/cloudimmunity/docker-slim/releases/download/1.14/dist_linux.zip) binaries. Unzip them and optionally add their directory to your PATH environment variable if you want to use the app from other locations.

	The extracted directory contains two binaries:

	* `docker-slim` <- the main application
	* `docker-slim-sensor` <- the sensor application used to collect information from running containers

1. Clone this repo to use the sample apps. You can skip this step if you have your own app.

	`git clone https://github.com/cloudimmunity/docker-slim.git`

2. Create a Docker image for the sample node.js app in `sample/apps/node`. You can skip this step if you have your own app.

	`cd docker-slim/sample/apps/node`

	`eval "$(docker-machine env default)"` <- optional (depends on how Docker is installed on your machine); if the Docker host is not running you'll need to start it first: `docker-machine start default`; see the `Docker connect options` section for more details.

	`docker build -t my/sample-node-app .`

3. Run `docker-slim`:

	`./docker-slim build --http-probe my/sample-node-app` <- run it from the location where you extraced the docker-slim binaries (or update your PATH env var to include the `docker-slim` bin directory)

	DockerSlim creates a special container based on the target image you provided. It also creates a resource directory where it stores the information it discovers about your image: `<docker-slim directory>/.images/<TARGET_IMAGE_ID>`.

4. Use curl (or other tools) to call the sample app (optional)

	`curl http://<YOUR_DOCKER_HOST_IP>:<PORT>`

	This is an optional step to make sure the target app container is doing something. Depending on the application it's an optional step. For some applications it's required if it loads new application resources dynamically based on the requests it's processing.

	You can get the port number either from the `docker ps` or `docker port <CONTAINER_ID>` commands. The current version of DockerSlim doesn't allow you to map exposed network ports (it works like `docker run … -P`).

	If you set the `http-probe` flag then `docker-slim` will try to call your application using HTTP/HTTPS: `./docker-slim build --http-probe my/sample-node-app`

5. Press <enter> and wait until `docker-slim` says it's done

6. Once DockerSlim is done check that the new minified image is there

	`docker images`

	You should see `my/sample-node-app.slim` in the list of images. Right now all generated images have `.slim` at the end of its name.

7. Use the minified image

	`docker run --name="slim_node_app" -p 8000:8000 my/sample-node-app.slim`

## USAGE DETAILS

`docker-slim [global options] command [command options] <Docker image ID or name>`

Commands:

* `build` - Collect fat image information and build a slim image from it
* `profile` - Collect fat image information and generate a fat container report
* `info` - Collect fat image information and reverse engineers its Dockerfile (no runtime container analysis)

Global options:

* ` --debug` - enable debug logs
* `--host` - Docker host address
* `--tls` - use TLS connecting to Docker
* `--tls-verify` - do TLS verification
* `--tls-cert-path` - path to TLS cert files

### `BUILD` COMMAND OPTIONS

* `--http-probe` - enables HTTP probing (disabled by default)
* `--http-probe-cmd` - additional HTTP probe command [zero or more]
* `--http-probe-cmd-file` - file with user defined HTTP probe commands
* `--show-clogs` - show container logs (stdout and stderr)
* `--remove-file-artifacts` - remove file artifacts when command is done (note: you'll loose autogenerated Seccomp and Apparmor profiles)
* `--tag` - use a custom tag for the generated image (instead of the default: `<original_image_name>.slim`)
* `--entrypoint` - override ENTRYPOINT analyzing image
* `--cmd` - override CMD analyzing image
* `--mount` - mount volume analyzing image (the mount parameter format is identical to the `-v` mount command in Docker) [zero or more]
* `--include-path` - Include directory or file from image [zero or more]
* `--continue-after` - Select continue mode: enter | signal | probe | timeout or numberInSeconds (default: enter)

The `--include-path` option is useful if you want to customize your minified image adding extra files and directories. Future versions will also include the `--exclude-path` option to have even more control.

The `--continue-after` option is useful if you need to script `docker-slim`. If you pick the `probe` option then `docker-slim` will continue executing the build command after the HTTP probe is done executing. If you pick the `timeout` option `docker-slim` will allow the target container to run for 60 seconds before it will attempt to collect the artifacts. You can specify a custom timeout value by passing a number of seconds you need instead of the `timeout` string. If you pick the `signal` option you'll need to send a USR1 signal to the `docker-slim` process.

## DOCKER CONNECT OPTIONS

If you don't specify any Docker connect options `docker-slim` expects to find the following environment variables: `DOCKER_HOST`, `DOCKER_TLS_VERIFY` (optional), `DOCKER_CERT_PATH` (required if `DOCKER_TLS_VERIFY` is set to `"1"`)

On Mac OS X you get them when you run `eval "$(docker-machine env default)"` or when you use the Docker Quickstart Terminal.

If the Docker environment variables are configured to use TLS and to verify the Docker cert (default behavior), but you want to disable the TLS verification you can override the TLS verification behavior by setting the `--tls-verify` to false:

`docker-slim --tls-verify=false build --http-probe=true my/sample-node-app-multi`

You can override all Docker connection options using these flags: `--host`, `--tls`, `--tls-verify`, `--tls-cert-path`. These flags correspond to the standard Docker options (and the environment variables).

If you want to use TLS with verification:

`docker-slim --host=tcp://192.168.99.100:2376 --tls-cert-path=/Users/youruser/.docker/machine/machines/default --tls=true --tls-verify=true build --http-probe=true my/sample-node-app-multi`

If you want to use TLS without verification:

`docker-slim --host=tcp://192.168.99.100:2376 --tls-cert-path=/Users/youruser/.docker/machine/machines/default --tls=true --tls-verify=false build --http-probe=true my/sample-node-app-multi`

If the Docker environment variables are not set and if you don't specify any Docker connect options `docker-slim` will try to use the default unix socket.

## HTTP PROBE COMMANDS

If you enable the HTTP probe it will default to running `GET /` with HTTP and then HTTPS on every exposed port. You can add additional commands using these two options: `--http-probe-cmd` and `--http-probe-cmd-file`.

The `--http-probe-cmd` option is good when you want to specify a small number of simple commands where you select some or all of these HTTP command options: protocol, method (defaults to GET), resource (path and query string).

Here are a couple of examples:

Adds two extra probe commands: `GET /api/info` and `POST /submit` (tries http first, then tries https):
`docker-slim build --show-clogs --http-probe-cmd /api/info --http-probe-cmd POST:/submit my/sample-node-app-multi`

Adds one extra probe command: `POST /submit` (using only http):
`docker-slim build --show-clogs --http-probe-cmd http:POST:/submit my/sample-node-app-multi`

The `--http-probe-cmd-file` option is good when you have a lot of commands and/or you want to select additional HTTP command options.

Here's an example:

`docker-slim build --show-clogs --http-probe-cmd-file probeCmds.json my/sample-node-app-multi`

Commands in `probeCmds.json`:

```
{
  "commands":
  [
   {
     "resource": "/api/info"
   },
   {
     "method": "POST",
     "resource": "/submit"
   },
   {
     "procotol": "http",
     "resource": "/api/call?arg=one"
   },
   {
     "protocol": "http",
     "method": "POST",
     "resource": "/submit2"
   }
  ]
}
```

The HTTP probe command file path can be a relative path (relative to the current working directory) or it can be an absolute path.


## MINIFYING COMMAND LINE TOOLS

Unless the default CMD instruction in your Dockerfile is sufficient you'll have to specify command line parameters when you execute the `build` command in DockerSlim. This can be done with the `--cmd` option.

Other useful command line parameters:

* `--show-clogs` - use it if you want to see the output of your container.
* `--mount` - use it  to mount a volume when DockerSlim inspects your image.
* `--entrypoint` - use it if you want to override the ENTRYPOINT instruction when DockerSlim inspects your image.

Note that the `--entrypoint` and `--cmd` options don't override the `ENTRYPOINT` and `CMD` instructions in the final minified image.

Here's a sample `build` command:

`docker-slim build --show-clogs=true --cmd docker-compose.yml --mount $(pwd)/data/:/data/ dslim/container-transform`

It's used to minify the `container-transform` tool. You can get the minified image from [`Docker Hub`](https://hub.docker.com/r/dslim/container-transform.slim/).

## CURRENT STATE

It works pretty well with the sample Node.js, Python (2 and 3), Ruby and Java images (built from `sample/apps`). More testing needs to be done to see how it works with other images. Rails/unicorn app images are not fully supported yet (WIP).

Sample images (built with the standard Ubuntu 14.04 base image):

* nodejs app container: 431.7 MB => 14.22 MB
* python app container: 433.1 MB => 15.97 MB
* ruby app container:   406.2 MB => 13.66 MB
* java app container:   743.6 MB => 100.3 MB (yes, it's a bit bigger than others :-))

You can also run `docker-slim` in the `info` mode and it'll generate useful image information including a "reverse engineered" Dockerfile.

DockerSlim now also generates Seccomp (usable) and AppArmor (WIP) profiles for your container.

Works with Docker 1.8, 1.9 and 1.10.

Note:

You don't need Docker 1.10 to generate Seccomp profiles, but you do need it if you want to use the generated profiles.

## FAQ

### Is it safe for production use?

Yes! Either way, you should test your Docker images.

### How can I contribute if I don't know Go?

You don't need to read the language spec and lots of books :-) Go through the [Tour of Go](https://tour.golang.org/welcome/1) and optionally read [50 Shades of Go](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/) and you'll be ready to contribute!

### What's the best application for DockerSlim?

DockerSlim will work for any dockerized application; however, DockerSlim automates app interactions for applications with an HTTP API. You can use DockerSlim even if your app doesn't have an HTTP API. You'll need to interact with your application manually to make sure DockerSlim can observe your application behavior.

### Can I use DockerSlim with dockerized command line tools?

Yes. The --cmd, --entrypoint, and --mount options will help you minify your image. The `container-transform` tool is a good example.

Notes:

You can explore the artifacts DockerSlim generates when it's creating a slim image. You'll find those in `<docker-slim directory>/.images/<TARGET_IMAGE_ID>/artifacts`. One of the artifacts is a "reverse engineered" Dockerfile for the original image. It'll be called `Dockerfile.fat`.

If you'd like to see the artifacts without running `docker-slim` you can take a look at the `sample/artifacts` directory in this repo. It doesn't include any image files, but you'll find:

*	a reverse engineered Dockerfile (`Dockerfile.fat`)
*	a container report file (`creport.json`)
*	a sample AppArmor profile (which will be named based on your original image name)
*   and a sample Seccomp profile

If you don't want to create a minified image and only want to "reverse engineer" the Dockerfile you can use the `info` command.

## BUILD PROCESS

Go 1.5.1 or higher is required. Earlier versions of Go have a Docker/ptrace related bug (Go kills processes if your app is PID 1). When the 'monitor' is separate from the 'launcher' process it will be possible to user older Go versions again.

Before you build the tool you need to install GOX and Godep (optional; you'll need it only if you have problems pulling the dependencies with vanilla `go get`)

* Godep - dependency manager ( https://github.com/tools/godep )

1: `go get github.com/tools/godep`

* GOX - to build Linux binaries on a Mac ( https://github.com/mitchellh/gox ):

1: `go get github.com/mitchellh/gox`

2: `gox -build-toolchain -os="linux" -os="darwin"` (note:  might have to run it with `sudo`)

Note:

Step 2 is not necessary with Go 1.5.

#### Local Build Steps

Once you install the dependencies (GOX - required; Godep - optional) run these scripts:

1. Pull the dependencies: `./scripts/src.deps.get.sh`
2. Build it: `./scripts/src.build.sh`

You can use the clickable `.command` scripts on Mac OS X (located in the `scripts` directory):

1. `mac.src.deps.get.command`
2. `mac.src.build.command`

#### Traditional Go Way to Build

If you don't want to use the helper scripts you can build `docker-slim` using regular go commands:

1. `cd $GOPATH`
2. `mkdir -p src/github.com/cloudimmunity`
3. `cd $GOPATH/src/github.com/cloudimmunity`
4. `git clone https://github.com/cloudimmunity/docker-slim.git` <- if you decide to use `go get` to pull the `docker-slim` repo make sure to use the `-d` flag, so Go doesn't try to build it
5. `cd docker-slim`
6. `go get -d -v ./...`
7. `go build -v ./apps/docker-slim` <- builds the main app in the repo's root directory
8. `env GOOS=linux GOARCH=amd64 go build -v ./apps/docker-slim-sensor` <- builds the sensor app (must be built as a linux executable)

#### Builder Image Steps

You can also build `docker-slim` using a "builder" Docker image. The helper scripts are located in the `scripts` directory.

1. Create the "builder" image: `./docker-slim-builder.build.sh` (or click on `docker-slim-builder.build.command` if you are using Mac OS X)
2. Build the tool: `docker-slim-builder.run.sh` (or click on `docker-slim-builder.run.command` if you are using Mac OS X)

## DESIGN

### CORE CONCEPTS

1. Inspect container metadata (static analysis)
2. Inspect container data (static analysis)
3. Inspect running application (dynamic analysis)
4. Build an application artifact graph
5. Use the collected application data to build small images
6. Use the collected application data to auto-generate various security framework configurations.

### DYNAMIC ANALYSIS OPTIONS

1. Instrument the container image (and replace the entrypoint/cmd) to collect application activity data
2. Use kernel-level tools that provide visibility into running containers (without instrumenting the containers)
3. Disable relevant namespaces in the target container to gain container visibility (can be done with runC)

### SECURITY

The goal is to auto-generate Seccomp, AppArmor, (and potentially SELinux) profiles based on the collected information.

* AppArmor profiles
* Seccomp profiles

### CHALLENGES

Some of the advanced analysis options require a number of Linux kernel features that are not always included. The kernel you get with Docker Machine / Boot2docker is a great example of that.


## DEVELOPMENT PROGRESS

### PHASE 3 (WIP)

* Auto-generate AppArmor profiles (almost usable :-))
* Option to pause builder execution to allow manual changes to the minified image artifacts.
* Support additional command line parameters to specify CMD, VOLUME, ENV info.
* Better support for command line applications

### TODO

* Discover HTTP endpoints to make the HTTP probe more intelligent.
* Scripting language dependency discovery in the "scanner" app.
* Explore additional dependency discovery methods.
* Build/use a custom Boot2docker kernel with every required feature turned on.
* "Live" image create mode - to create new images from containers where users install their applications interactively.

## ORIGINS

DockerSlim was a [Docker Global Hack Day \#dockerhackday](https://www.docker.com/community/hackathon) project. It barely worked at the time :-)

Since then it's been improved and it works pretty well for its core use cases. It can be better though. That's why the project needs your help! You don't need to know much about Docker and you don't need to know anything about Go. You can contribute in many different ways. For example, use DockerSlim on your images and open a Github issue documenting your experience even if it worked just fine :-)

## ONLINE

IRC (freenode): \#dockerslim

Docker Hub: [dslim](https://hub.docker.com/r/dslim/) (dockerslim is already taken :-()

## MINIFIED DOCKER HUB IMAGES

* [`container-transform`](https://hub.docker.com/r/dslim/container-transform.slim/)

## NOTES

* The code is still not very pretty at this point in time :)

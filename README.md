# dockersnips
A collection of dockerfile snippets and useful commands.

In poking around for decent pages on building Docker images, I came across [this document.][1] I don't know how up-to-date it is, but it's pretty comprehensive, with a lot of interesting scenarios e.g. an image for doing Firefox development, an image hosting a graphical environment that you can "remote into" from your host environment.

## Building arbitrary docker files

I'm creating multiple Dockerfiles in this folder. By default/convention, `docker` looks for a file named Dockerfile at whatever path you give it, e.g:

```
$ docker build .
```

Unless there's a file named `Dockerfile` in the current directory, `docker` will complain. To specify a specific arbitrary file name, as I will with this project, you need to pipe in the contents of the file via `STDIN`:

```
$ docker build -t <tagname> - < <docker file name>
```

For instance, to build my base image, I issued the command, with the following response:

```
$ docker build -t mbbase - < Dockerfile.aa_baseimage
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM busybox
 ---> 2b8fd9751c4c
Step 2 : MAINTAINER Bob Campbell mostlybob@gmail.com
 ---> Using cache
 ---> da4b4fcd6b70
Successfully built da4b4fcd6b70
```

The `-t <tagname>` switch is optional, but it's probably a good habit to have when your `docker images` command starts showing dozens(!) of images. I may need to brush up my housekeeping skills.

## Starting and Attaching to a container

I have to admit, I'm still learing about running my Docker stuff "without a net," so much of this can probably be done in a simpler or more efficient manner, but this is what I've been finding.

After building the file, presumably with a tag for easy reference, I run it with:

```
$ docker run -dit <TAG> | <IMAGE ID>
```

The `-dit` switch, which you can look up by issuing `docker help run`, means, in order, detached, interactive, use a pseudo or virtual tty. That last one allows you to attach to the running container and see your handiwork.

Using my previous example, I issued the following command, with this response:

```
$ docker run -dit mbbase
f1b7a486144cc74685af2b378e3bb62d6d4885afcd636d116421d5c563b6db04
```

This is rather obscure, but what's returned by the command is the id docker uses to identify the container. Fortunately, there are other ways to refer to the container. Run `docker ps` to see the running containers:

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
f1b7a486144c        mbbase              "sh"                4 seconds ago       Up 3 seconds                            ecstatic_noether
```

Docker uses an abbreviated form of the long id returned by the `docker run` command. You can use that value or the easier to type name that `docker` assigns the container, in this case `ecstatic_noether`. Docker has the quirky pattern of naming containers with a combination of an adjective and the surname of a famous scientist. Sounds like a fun and relatively harmless project to give an intern. These names are ephemeral. If you kill the container and rerun it, the name will be different. Not surprisingly, so is the container id:


```
$ docker run -dit mbbase
f1b7a486144cc74685af2b378e3bb62d6d4885afcd636d116421d5c563b6db04

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
f1b7a486144c        mbbase              "sh"                4 seconds ago       Up 3 seconds                            ecstatic_noether

$ docker kill ecstatic_noether 
ecstatic_noether

$ docker run -dit mbbase
939639d672153f53b2a2e207e7d6d061e940216d8881155d46f12b5398ed0152

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
939639d67215        mbbase              "sh"                6 seconds ago       Up 5 seconds                            evil_brown
```


-----

[1]: https://docs.google.com/document/d/1f8iflnFSZxAU9FhoLQPEVlSKhVPXbtCaqTVPTTJb9yo/edit#heading=h.py35px4li4o2


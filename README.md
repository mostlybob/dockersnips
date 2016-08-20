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

-----

[1]: https://docs.google.com/document/d/1f8iflnFSZxAU9FhoLQPEVlSKhVPXbtCaqTVPTTJb9yo/edit#heading=h.py35px4li4o2


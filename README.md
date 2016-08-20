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

Attaching to a running container allows you to, in a sense, `ssh` into the container and interact with the running instance. This is another area where I've had mixed results. In some other tutorial projects, I found this method would not attach to the container. My bet is that it has to do with the running process. For the purposes of this document, I intentionally created a simple container that's running `sh`, so I can easily attach to it. This one of those areas that will probably develop as my experience with Docker grows.

> As with all `docker` commands the help for the command can be viewed with `docker help <command>`. As with a lot of these styled help messages, they're more pointers for further reference or a shorthand reminder for the more experienced among us.

Issue the command:

```
$ docker attach <NAME> | <IMAGE ID> 
```

Using my example above, I get this:

```
$ docker attach evil_brown 
/ # 
```

That `/ #` is the command prompt running inside the container. The `#` indicates that within the container, you're running as root. By default, Docker container processes run with root privileges within the container. _(Actually, I'm not sure you can create a sub-root user to run inside the process. Probably, but I haven't seen or had that need yet.)_

Let's take a look.

```
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # pwd
/
```

Running `top` shows a pretty spartan environment:

```
Mem: 5190120K used, 2678612K free, 569044K shrd, 8432K buff, 768016K cached
CPU:  1.2% usr  0.4% sys  0.0% nic 97.5% idle  0.8% io  0.0% irq  0.0% sirq
Load average: 0.27 1.98 1.51 2/951 8
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 root     S     1196  0.0   2  0.0 sh
    8     1 root     R     1188  0.0   2  0.0 top
```

Getting out of the interactive session is one area where I've had to be careful. Normally when running a root session, I would type `exit` to get out, but see what happens:

```
/ # exit
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

What happened to the container? I learned the hard way that `exit` actually kills the container. Start it back up and let's do it properly:

```
$ docker run -dit mbbase
4c37602a8b3e85039a26f5af4a704b38fd9b20a46c8e820f8635b34fec7c2f71

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4c37602a8b3e        mbbase              "sh"                4 seconds ago       Up 3 seconds                            berserk_knuth

$ docker attach berserk_knuth 

/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var

/ # 

/ # $ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4c37602a8b3e        mbbase              "sh"                3 minutes ago       Up 3 minutes                            berserk_knuth

$
```

Wait, what? Yeah, me too.

I found a handy [SO post][2] which says that, from within the running container (our `/ #` prompt) issue the command `Ctrl-p Ctrl-q` to drop out to the host prompt and leave the container running. Easy!

-----

[1]: https://docs.google.com/document/d/1f8iflnFSZxAU9FhoLQPEVlSKhVPXbtCaqTVPTTJb9yo/edit#heading=h.py35px4li4o2
[2]: https://stackoverflow.com/questions/19688314/how-do-you-attach-and-detach-from-dockers-process


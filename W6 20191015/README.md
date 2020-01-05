# W6 20191015

- [W6 20191015](#w6-20191015)
- [Docker compose](#docker-compose)
  - [Dockerfile](#dockerfile)
    - [What's Dockerfile &amp; Why use it?](#whats-dockerfile-amp-why-use-it)
    - [Dockerfile components](#dockerfile-components)
    - [Dockerfile example](#dockerfile-example)
    - [Dockerfile example 2](#dockerfile-example-2)
    - [Dockerfile example 3](#dockerfile-example-3)
  - [Reference](#reference)

# Docker compose

- def: Compose is a tool for defining and running multi-container Docker applications
- by add a YAML file to configures

---

## Dockerfile

> ref:
> - [Learn Enough Docker to be Useful - Towards Data Science](https://towardsdatascience.com/learn-enough-docker-to-be-useful-b7ba70caeb4b)
> - [Docker - Dockerfile 指令教學，含範例解說 - 靖．技場](https://www.jinnsblog.com/2018/12/docker-dockerfile-guide.html)

- 2 ways to create Docker image
  1. Download the base image, then run the container with this image (metioned before)
  2. Dockerfile (today's topic)

### What's Dockerfile & Why use it?

- write all the info into the docker file, then it can build a custom image
- more secure: we can know the components of this image
- slim size: it's just a text file, easy to share

### Dockerfile components

- `#`: comments
- `FROM <image>:<tag>`: base image
  - if we don't specify tag, it will using the latest version
- `MAINTAINER <name>`: author
- `LABEL <key>=<value> <key>=<value>...`
  - we can use `docker inspect` to search label info
- `RUN`
  - there're two methods to write `RUN` command
    1. shell format: `RUN program param1`
    2. exec format: `RUN ["program", "param1", "param2"]`
  - we can use `\` to line feed
- `CMD`
- `WORKDIR`: change the working directory
- `COPY`: copy local file to container
- `expose`:

- Q: What's the difference between `RUN` and `CMD`
  - `CMD` just like `docker run --itd <image> ifconfig`

### Dockerfile example

- for the performance reason, we will try our best to cut down the number of layers
- thus we use `&&` to compound multiple commands

```
# Dockerfile
FROM ubuntu:latest
MAINTAINER minidino istar0me@gmail.com
RUN mkdir -p /home/demo/docker
RUN apt-get update && apt-get install -y apache2 python3
```

- Now it's time to build our image from Dockerfile

```
[root@localhost ~]# docker build -t myimage:v0.1 .
```

> - If Dockerfile file name isn't `Dockerfile` (e.g. `dfile`), then we have to specify the name
> 
> ```
> docker build -t myimage:v0.1 -f dfile
> ```

```
[root@localhost ~]# docker run -it --name c10 --rm myimage:v0.1 bash
root@648d0513e2fa:/# 
```

---

### Dockerfile example 2

```
# Dockerfile2
FROM centos
MAINTAINER minidino istar0me@gmail.com
RUN yum update -y && yum install -y httpd net-tools
EXPOSE 80
```

- build & run

```
[root@localhost ~]# docker build -t myimage:v0.2 . -f Dockerfile2
# hide the build process
[root@localhost ~]# docker run -it --name c11 -p 8080:80 myimage:v0.2 bash
[root@c129b8df63a4 /]# /usr/sbin/httpd -f /etc/httpd/conf/httpd.conf
 # initialize httpd server daemon
```

- `docker inspect <containerName>` to see the all the info about the container

```
docker inspect c11 | grep IP
```

```
echo "hi" > /var/www/html/hi.html # run in the container
curl 127.0.0.1:8080/hi.html # run in the container
curl <containerIP>:8080/hi.html # run in the host
```

---

- we can copy our files from host machine
  - note: don't forget to add hello.html file

```
[root@localhost ~]# echo "hello" > hello.html
```

```
# Dockerfile2.1
FROM centos
MAINTAINER minidino istar0me@gmail.com
RUN yum update -y && yum install -y httpd net-tools
EXPOSE 80
COPY hello.html /var/www/html
```

- after modifying the Dockerfile, don't forget to build again

```
docker build -t myimage:v0.2 . -f Dockerfile2.1
docker run -it --name c21 -p 8080:80 myimage:v0.2 bash
/usr/sbin/httpd -f /etc/httpd/conf/httpd.conf # initialize httpd server daemon
```

- test

```
[root@1e8135b835c0 /]# curl 127.0.0.1:80/hello.html
hello
```

### Dockerfile example 3

- If the container process run in the backgroun, the container will closed automatically
- thus, we have to run our program in the foreground

```
# Dockerfile3
FROM centos
MAINTAINER minidino istar0me@gmail.com
RUN yum update -y && yum install -y httpd net-tools
COPY hello.html /var/www/html
EXPOSE 80
CMD ["/usr/sbin/httpd", "-DFOREGROUND"] # force run in the foreground
```

```
docker build -t myimage:v0.4 . -f Dockerfile3
docker run -itd -p 8080:80 myimage:v0.4
curl 127.0.0.1:8080/hello.html
```

## Reference

1. [Overview of Docker Compose | Docker Documentation](https://docs.docker.com/compose/)
2. [Get started with Docker Compose | Docker Documentation](https://docs.docker.com/compose/gettingstarted/)
3. [Quickstart: Compose and WordPress | Docker Documentation](https://docs.docker.com/compose/wordpress/)
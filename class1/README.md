# Before You Start
You must have installed your docker-engine environment to start this tutorial.

# Terminology
`Docker Hub` -- A public directory of docker images. If you set up a private directory of yourself, then it is called 'docker registry'.
`Docker daemon` -- A process running on your local machine/server, that do the miracle of docker.
`Docker client` -- A program that can interact with Docker daemon, that human user can send commands to daemon and receive response.
`Docker image` -- It is like a virtual disk of a virtual machine. It contains operating system and the programs on the operating system.
`Docker container` -- A live, running virtual machine. It is created from a docker image. You can run the container in background, or use interactive mode to work inside a container.

# Important
You have to run the commands in this tutorial in `root` previlige. Otherwise you need to create a user group that can run `docker` command.

# Check Docker version and Linux Kernel Version 
```
root@JapanBackup:~# docker version
Client:
 Version:      1.12.6
 API version:  1.24
 Go version:   go1.6.4
 Git commit:   78d1802
 Built:        Tue Jan 10 20:38:45 2017
 OS/Arch:      linux/amd64

Server:
 Version:      1.12.6
 API version:  1.24
 Go version:   go1.6.4
 Git commit:   78d1802
 Built:        Tue Jan 10 20:38:45 2017
 OS/Arch:      linux/amd64

root@JapanBackup:~# uname -r
4.8.0-30-generic
```

From the above commands, we can see that the `docker client` program we use as `docker` command is on version `1.12.6`, the `docker daemon` we use is on version `1.12.6`. Our linux kernel version is on `4.8.0`.

# Pull some images and check images in docker

```
# We pull an operating system, a lightweight linux called alpine from registry
root@JapanBackup:~# docker pull alpine
```

Now `alpine` image is in our local images inventory. We try to check it out.

```
# We check the images on our system, the repo name is alpine, the version is latest. it has a size of 3.984 MB
root@JapanBackup:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              88e169ea8f46        2 weeks ago         3.984 MB
```

# Start an container, and execute an command in container
```
# Command format: docker run [image name] [command] [command args]

# Check who is the user we log in as in the container
root@JapanBackup:~# docker run alpine whoami
root

# Check which entry directory we are in as in the container
root@JapanBackup:~# docker run alpine pwd
/

# Create a file namex abc.txt
root@JapanBackup:~# docker run alpine touch abc.txt

# Check in the entry directory, what files are in the directory
root@JapanBackup:~# docker run alpine ls -l
total 52
drwxr-xr-x    2 root     root          4096 Dec 26 21:32 bin
drwxr-xr-x    5 root     root           360 Jan 15 09:30 dev
drwxr-xr-x   14 root     root          4096 Jan 15 09:30 etc
drwxr-xr-x    2 root     root          4096 Dec 26 21:32 home
drwxr-xr-x    5 root     root          4096 Dec 26 21:32 lib
drwxr-xr-x    5 root     root          4096 Dec 26 21:32 media
drwxr-xr-x    2 root     root          4096 Dec 26 21:32 mnt
dr-xr-xr-x  147 root     root             0 Jan 15 09:30 proc
drwx------    2 root     root          4096 Dec 26 21:32 root
drwxr-xr-x    2 root     root          4096 Dec 26 21:32 run
drwxr-xr-x    2 root     root          4096 Dec 26 21:32 sbin
drwxr-xr-x    2 root     root          4096 Dec 26 21:32 srv
dr-xr-xr-x   13 root     root             0 Jan 15 09:30 sys
drwxrwxrwt    2 root     root          4096 Dec 26 21:32 tmp
drwxr-xr-x    7 root     root          4096 Dec 26 21:32 usr
drwxr-xr-x   12 root     root          4096 Dec 26 21:32 var
```

As we see above, We are logging into the fresh created container with the user `root` and inside the directory `/`. We can create a file `abc.txt` inside the directory, but when we run again and check the directory, it is not there. Why? Because each time we do a 'docker run' command, it will create a new container, and the contaner will be `stop` status once the command you supplied to run is exited.

# Check runnning containers and stopped containers
```
# Command format: docker ps [-a]
# Check running containers
root@JapanBackup:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
# Check all containers, including stopped containers
root@JapanBackup:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
332100b9a2b8        alpine              "ls -l"             7 minutes ago       Exited (0) 7 minutes ago                        pedantic_wright
e764abab6365        alpine              "touch abc.txt"     7 minutes ago       Exited (0) 7 minutes ago                        happy_swartz
0140ef8cba19        alpine              "whoami"            9 minutes ago       Exited (0) 9 minutes ago                        focused_spence
d005e965505b        alpine              "pwd"               9 minutes ago       Exited (0) 9 minutes ago                        grave_pare
a3e14e66d599        alpine              "ls -l"             10 minutes ago      Exited (0) 10 minutes ago                       mad_kare
```

We can see all containers are stopped after the command we executed is finished, so `docker ps` gets us nothing. If we tell it to give us back all the containers we have launched and stopped, we add `-a` switch. We can see docker automatically gives a new name to container, like grave_pare or so, and it will give each container an ID like `332100b9a2b8`.

# Run container as a Interactive Mode
```
# Use -i to be interactive mode, use -t to attach a tty console to the container
# We run the first command in container as sh.
root@JapanBackup:~# docker run -i -t alpine sh
/ # ps
PID   USER     TIME   COMMAND
    1 root       0:00 sh
    5 root       0:00 ps
/ # pwd
/
/ # whoami
root
/ # touch abc.txt
/ # ls
abc.txt  bin      dev      etc      home     lib      media    mnt      proc     root     run      sbin     srv      sys      tmp      usr      var
/ # exit
```

We run a container using the alpine image, then we use `i` and `t` flag to indicate we want to run into interactive mode. We execute some command and exit the container.


# Quick Tips
How to show containers only with ID, without the long input?
```
root@JapanBackup:~# docker ps -a -q
0a928589cd0c
9f7d30f9ec54
332100b9a2b8
e764abab6365
0140ef8cba19
d005e965505b
a3e14e66d599
```

How to remove old containers, if they are stopped?
```
docker rm [container ID]
```

How to quickly remove old containers, if I don't want to type in container ID by hand?
```
docker rm $(docker ps -a -q)
```

Notice that if the container is still running, you cannot `rm` it. You have to `docker stop [container ID]` first.
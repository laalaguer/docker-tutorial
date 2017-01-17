# Create a Running Web Page Server
This tutorial we set up a web server that can serve a web page.

# Knowledge Prepare

## What is Nginx？
Nginx is a open source software and is aming at replacing Apache. They both serve as a web content provider server. Apache is all-mighty and heavy. Nginx is lightweight, has limited functions, but faster.

## How to Use Nginx?
We usuall use `nginx` to serve static contents, for example, `html, jpg, png, css` files. We need to configure two things in order for `nginx` to work, we need to provide

1. Content: html, png, jpg files that you want to server to the visitors.

2. Configuration(s) of nginx.

### Content
On Linux, `/usr/share/` folder is the place for optional data shared by users.
By default, `html` content goes into `/usr/share/nginx/html` folder.  We put the `index.html` there.
The folder sturcture looks like below:
```
/usr/share/nginx
-- index.html
```

### Nginx Configurations
On Linux, Nginx has configurations in `/etc/nginx/` folder.
Under this folder, `nginx.conf` is the main configuration file. It defines default Nginx configs.
The sub-folder `conf.d` folder, we put several user defined configuration files.
On Nginx starting phase, it will first check the main configuration, then will read the configurations in `conf.d` folder to check out user preferences. The user configs override the default main configuration.
The folder sturcture looks like below:
```
/etc/nginx
-- nginx.conf
-- conf.d/
```

# Explore the web server image
The previous discussed Linux image is already existing in the docker registry. It is called `seqvence/static-site`, let us pull it and explore it.
```
$ docker pull seqvence/static-site

$ docker run --name my-static-site -e AUTHOR="Francis Underwood"\
	 -p 8888:80 -it seqvence/static-site\ 
	 sh
# We use name flag to give our container a name, 
# Use -e to pass an environment variable, 
# Use -p to let host 8888 port mapping to container 80 port.
# Use -it to get an interactive terminal. 
# And finally, we override the default CMD 
# by telling the container to execute sh command. 
# So we log in and get a command line prompt.
```

Then we go into the container. Let's check the files in the container by:
```
$ ls /usr/share/nginx/html
50x.html  Hello_docker.html  index.html

$ ls /etc/nginx/
conf.d	nginx.conf fastcgi_params	koi-utf  koi-win  mime.types  modules  scgi_params	uwsgi_params  win-utf

$ ls /etc/nginx/conf.d
default.conf
```

If you further using `cat` command to list the content of `nginx.conf` and `default.conf`, you will see these configurations tell the nginx to serve the webpage of `/usr/share/nginx/index.html` publicly when visitor visit the container at 80 port.

As we have done exploring the file system of container, we simply type `exit` to get off. 
```
$ exit
```

## What is docker file?
Docker file is a file. It is a guide for docker daemon to build an image. You can think of it as a **starting script** to build an image, just like you prepare a raw server and install dependencies. This file mainly contains three parts: 
1. The base OS image it is based on.
2. The commands of installation to execute furthur on the image.
3. The default command that will execute if the user doesn't specify a command when he launches this image.

Let us see the example docker file of this static website: [Dockerfile](https://github.com/docker/labs/blob/master/beginner/static-site/)
```
FROM nginx
ENV AUTHOR=Docker

WORKDIR /usr/share/nginx/html
COPY Hello_docker.html /usr/share/nginx/html

CMD cd /usr/share/nginx/html && sed -e s/Docker/"$AUTHOR"/ Hello_docker.html > index.html ; nginx -g 'daemon off;'
```

The `FROM` indicates which base image we are building upon, here is `nginx` image (it is in online docker registry), then it assigns an linux env variable `AUTHOR`. It specifies the install commands are executed inside this `WORKDIR`, so the following commands are executed in the container, under the specified directory. After that, it copies `Hello_docker.html` to the container, copies to `/usr/share/nginx/html` directory. Finally, it specifies a `CMD` as a command to execute if the user launching the image doesn't speicify a command to do.

In the `CMD` command, it uses a `sed` command to subsitute the text `Docker` into the value of local variable `$AUTHOR` in Hello_docker.html, output the result to index.html. Then it executes the command `nginx -g 'daemon off;'` to let the nginx to run in the foreground. Hence the launched container will not exit after nginx starts, because normal nginx will spawn child process then return immediately after launching.

As we have discussed before, `nginx` program will go into its default configuration location: `/etc/nginx` to look for default configuration and user configurations, then it will start.


# Launch the webserver image, get running container

This time, we lauch a new container and let it execute the `CMD` it defined in default settings and see the result:
```
$ docker run --name my-static-site-again \
	-e AUTHOR="Benjamin Franklin" \
	-p 8888:80 \
	seqvence/static-site
# This time we didn't lauch in -it mode, we didn't specify the command to run. So container will execute default CMD specified in Dockerfile
```

Or if you prefer take your command prompt back, let container run in background:
```
$ docker run --name my-static-site-again \
	-e AUTHOR="Benjamin Franklin" \
	-p 8888:80 \
	-d \
	seqvence/static-site
# This time we didn't lauch in -it mode, we didn't specify the command to run. So container will execute default CMD specified in Dockerfile
```


Now we can visit our little website on your local machine `http://localhost:8888/`, or if you run on a remote server in cloud, by `http:/your-server-ip:8888/`
Use `docker port` to check your container's exposed ports to the host
```
$ docker port my-static-site
80/tcp -> 0.0.0.0:8888
```

# Clean up your containers

When you have done with playing our little website, you can use `docker ps -a` to list all the running/stopped containers, use `docker stop [container name]` to stop a container, and `docker rm [container name]` to wipe the container out from disk.

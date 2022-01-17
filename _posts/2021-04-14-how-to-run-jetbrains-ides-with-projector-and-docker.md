---
layout: post

title: How to run Jetbrains' IDEs with Projector and Docker
---
As described in the previous [post]({% post_url 2021-04-13-jetbrains-projector-docker-images %}), the Projector is
JetBrains’ solution to remote development. It’s not a perfect solution but it has it’s uses. In this post, we will see
how to use Docker to run the IDEs with Projector and how to connect to them.

Projector is a client server solution. We need to deploy the two components, client and server, manually and
independently. Thankfully, Docker simplifies the server side deployment a lot. Especially if you use the images I
maintain in DockerHub!

# Projector server

The projector server is to be deployed on the machine that will host the IDE, the code, the builds and maybe the
execution of the application you will develop through the IDE. So it could be a cloud instance, a headless server at
your workplace or something similar. The host should have Docker already installed. Follow
the [official guide](https://docs.docker.com/engine/install/) if you need help with that.

To start the IDE of your choice with Projector, connect to the shell of the server (via SSH probably) and issue the
following command:

```bash
docker run --rm -p 8887:8887 --name <container-name> projectorimages/<Image-Name>
```

Replace `<Image-Name>` with the name of the application you want and the `<container-name>` with an identifier of your
liking. This will download the latest stable version from DockerHub and start a container which will listen on port 8887
on the host. For example, to start IntelliJ IDEA Community in a container name idea-community:

```bash
docker run --rm -p 8887:8887 --name idea-community projectorimages/projector-idea-c
 ```

Find the list of available images and their versions in [DockerHub](https://hub.docker.com/u/projectorimages).

# Projector client

There are two client options, you may use your browser in the client (which may be a tablet or a Chromebook), or you may
download the Projector Client from JetBrains.

## Browser

Open a new tab in your browser and navigate to the address of the server you deployed above, at port 8887. So if your
server is at IP 192.168.1.3, type in the browser address bar: `http://192.168.1.3:8887`. This will connect to the
Projector and show you your familiar IDEA interface.

![IntelliJ IDEA in a browser](/assets/images/2021/04/14/projector-IDEA.png)

## Projector Client

JetBrains has developed a client for Projector based on [Electron](https://www.electronjs.org/). You may opt to install
it on you client machine as according to a [blog post](https://blog.jetbrains.com/blog/2021/03/11/projector-is-out/) by
the creators, it supports shortcuts that are unavailable through a browser. You may download the latest version for
Windows, Linux or Mac from [here](https://github.com/JetBrains/projector-client/releases), or though the JetBrains
toolbox if you have it installed.

![](/assets/images/2021/04/14/projector-client.png)

When you start the Projector client you are greeted with an input field and a large “Connect” button. You are supposed
to enter the HTTP address and port as you did in the browser to connect to the IDE.

![](/assets/images/2021/04/14/projector-client-connected.png)

When connected, the experience is as close to natively running IntelliJ as it gets right now.

# Saving state of Projector containers

When you stop the container we started at the beginning of this post, all the data you created with your IDE (code,
plugins downloaded, settings, licenses) will be lost. Of course this is not desirable as we want to quickly resume our
work after the server is restarted or when we update our IDE. We can use a folder of the server to store all the data of
the container and be able to mount it to future containers. So first create a directory (e.g. in your home folder) and
then use it to mount at the /home/projector-user location in the container.

```bash
mkdir ~/projector_data
docker run --rm -p 8887:8887 -v ~/projector_data:/home/projector-user --name idea-community projectorimages/projector-idea-c
```

# Running multiple IDEs on the same server

In many cases it is required to run multiple IDEs when developing. For example I frequently run PyCharm alongside
WebStorm for my backend and frontend. You can do that with Projector too and you can even use the same directory to save
data as the IDEs don’t work on the same files.

There’s only one thing to take care, bind each IDE to a different host port, it is not possible to have two containers
bound the same host port. So you may decide to bind PyCharm on 8887 and WebStorm on 8888. Here’s how to start the
containers:

```bash
docker run --rm -p 8887:8887 -v ~/projector_data:/home/projector-user --name pycharm-community projectorimages/projector-pycharm-c
docker run --rm -p 8888:8887 -v ~/projector_data:/home/projector-user --name webstorm projectorimages/projector-webstorm
```

# Working with multiple projects on the same IDE

This is the biggest problem with projector. When you want to work with two or more projects in one IDE, you are asked to
open each one in its own window. The same thing happens with Projector, but both IDE windows open in the same browser
tab or client instance. It is not possible to have a separate client for each IDE window. That means that you have to
switch windows in the same tab using the `Ctrl+Alt+[` and `Ctrl+Alt+]` shortcuts. Not to mention that many times windows
seem to get stuck one above the other. Or the fact that you cannot take advantage of your multi-screen setup.

You may be tempted to start a new projector server for each project that you want to open, but this itself has its own
drawbacks. You cannot use the same directory mount for both containers as the first one locks files and disallows access
to the next ones. You may use a different mount point but then you need to re-enter all your IDE settings, licenses and
plugins and also remember which mountpoint to use when you want to open the project again in the future. It’s really
cumbersome.

# Updating IDEs to a newer version

If you used the commands above, it implied that you downloaded the latest docker images available as the default tag is
“latest”. When a new version is available, you have to stop the running container, pull the latest image again and
restart it.

```bash
docker stop pycharm-community
docker pull projectorimages/projector-pycharm-c
docker run --rm -p 8887:8887 -v ~/projector_data:/home/projector-user --name pycharm-community projectorimages/projector-pycharm-c
```

Or you may use the tags available at DockerHub to have more control on the version you are running.

One thing to note, you shouldn’t download new versions when prompted by the IDE as these updates will be lost the next
time you restart the container.

# Conclusion

Projector as a solution for remote development has left me with mixed feelings. It is usable if you can live with the
shortcomings – no real multi-project support, occasional latency based on network condition, cumbersome and manual setup
and maintenance. It’s miles behind the one-click wonder of VSCode. But if you have learned to live with the goodies
JetBrains IDEs have over VSCode (Java support, automation of tasks, refactoring, plugins) it may be worth the effort
until something better is available.

If your remote server is not headless, I had much better experience running IntelliJ on the remote and using SSH
X-Forwarding to get the UI in my thin client.

Or if your client is strong enough to index and build your project and you need the remote server to only run/debug your
application, consider using the new (in
2021.1) [Run Targets](https://blog.jetbrains.com/idea/2021/01/run-targets-run-and-debug-your-app-in-the-desired-environment/)
feature which is promising.
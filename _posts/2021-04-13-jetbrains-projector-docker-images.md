---
layout: post
title: Jetbrains' Projector Docker images
---
In the world of IDEs JetBrains has some widely used solutions like IntelliJ IDEA and PyCharm. Microsoft with VSCode has
been steadily gaining traction the last few years as it’s a really nice product, albeit much simpler and reduced in
functionality compared to JetBrains’ products or Eclipse.

The pandemic that changed our workspace and habits the last year has seen VSCode exploding due to a killer feature;
remote development. This feature allows the user of the IDE to have it automatically install a server instance in a
remote server and use the local window as a frontend. This means that code development, indexing, builds and everything
happens remotely (over SSH) and the user is not involved in setting up complex setups to do this; he just works with the
IDE as if it was local development.

The use cases for this kind of setup are varying. Some companies require employees to develop on remote servers and not
check out code on their home computers for security reasons. Other people develop directly on cloud instances to
replicate the production environment. And other people like me, want to use their power-horse headless server to develop
because their measly laptop is not up to par.

So JetBrains has been loosing clients to VSCode and has been working on a feature
for [remote development](https://youtrack.jetbrains.com/issue/IDEA-226455) for quite a few months now and has provided
the Projector solution as an attempt to remedy the problem.

[Projector](https://github.com/JetBrains/projector-server) is a server that can run any Java Swing application and
“project” the UI over HTTP(S) to a remote client. All JetBrains IDEs happen to be Swing applications so there you have
it. The remote client may be a browser or an Electron app developed by JetBrains called Projector Client.

You can install the Projector server as a plugin to the IDE and connect to it, but this doesn’t cover most of the use
cases where the server is headless (like an AWS instance). Projector can run headless of course but then the user needs
to manually install it in the remote machine, connect it with the IDE and start it before connecting to it. It’s not
very streamlined yet as you can understand.

Thankfully JetBrains has created a project to package their IDEs
and [projector as Docker containers](https://github.com/JetBrains/projector-docker) which greatly eases the process. But
for some strange reason they don’t upload the container images in DockerHub and they don’t even maintain their privately
hosted images. The current IDEA image is several months old at version 2020.2.1 while the current latest is 2021.1. They
have an [issue](https://youtrack.jetbrains.com/issue/PRJ-284) about that, but no indication of resolution.

Thus I decided to create a semi-automatic process that will use the projects created by JetBrains and will build and
upload to DockerHub images for all their IDEs, the [Jet-Builder](https://github.com/projectorimages/jet-builder). This
project uses git tags to trigger different build jobs in DockerHub. All the created images can be
found [here](https://hub.docker.com/u/projectorimages). I believe I will be able to trigger the build job soon after new
versions are released until I make a scrapper to collect their release announcements automatically.

- IntelliJ IDEA Community
- PyCharm Community
- IntelliJ IDEA Ultimate
- PyCharm Professional
- WebStorm
- Rubymine
- PhpStorm
- Rider
- Datagrip
- GoLand
- CLion

You can use
the [instructions](https://github.com/JetBrains/projector-docker/blob/12ae87c11899a69b89b5fd5cb636d8e4ae2b18ad/README.md)
provided by JetBrains to run these projector images, replacing their images with the ones above. Or wait for my next
blog post!
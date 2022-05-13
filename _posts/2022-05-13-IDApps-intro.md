---
layout: post

title: Introducing Instant Docker Apps
---
## The wealth of tools

As developers, we've all been in the situation where we need to set up external tooling to deploy and test our application.
Be it a database, a message queue, a monitoring system and what-not.

If you're a bit like me, you don't like installing a whole lot of stuff like these on your system; they are used for a
while and then are forgotten, but each time your laptop starts it tries to boot postgres, mysql, elasticsearch, rabbitmq,
redis, pgadmin and several others. And by the time you need them again, they are outdated and you have forgotten the
credentials and how to reconfigure them.

Same goes for command line tools. You downloaded aws-cli, used it for that client, the next client was on Azure, the
third on some obscure minor provider, you got all the CLIs. And Terraform, helm, kubectl, oc and the list goes on. Then
you change your laptop, or create a new cloud instance, and you need to start installing them all again as you need them.

Obviously Docker was a boon and a solution to all these problems. You start the db container, test your application and
then it's over. No data left over, no databases running in the background if you don't need them. And when you need it
again, you just pull the latest version from DockerHub. Sweet!

But this can be improved too. Each service requires some configuration, maybe some storage, mounting folders, taking
care of permissions. I find myself on the DockerHub documentation pages for postgres and nats all the time. I don't need
to recall for the hundredth time, where to mount persistent storage for MariaDB, I just want the database, and I want it
now. So, I started creating scripts to deploy these containers automatically.

Then I though that I cannot be the only one with this problem, surely most devs must face it. Someone must have solved it
by providing an easy-to-use collection of such scripts. I searched but to no avail. So here are mine for anyone that wants
to use them. I call them [IDApps](https://github.com/BabisK/idapps).

## Instant Docker Apps

Is it revolutionary like Docker? Not even close. Is it innovative? It uses 30+ year old technology. Is it useful? Hell
yes!

IDApps is a curated collection of simple shell scripts to invoke commonly used (in the dev/devops world) tools through
docker without any configuration. You want a postgres server? Here it is:

```bash
$ 
postgres started on localhost, listening on port 5432, username: postgres, password: postgres
```

Do you also need the pgadmin to view the database? Just call it:

```bash
$ 
pgadmin4 started at http://localhost:8080, username: user@pgadmin.org, password: pgadmin
```

That's it. If you need to see the logs of the service call:

```
$ postgres logs
```

If you don't need it anymore, invoke:

```
$ postgres stop
postgres has stopped
```

And it's gone. For good. If you ever need it again, call `postgres` and it will start again and use the data you had from
the previous run. If you don't want the data just call:

```
$ postgres clean
Cleaned up data for postgres
```

And the data are removed. As simple as that.

## What it is

I have set 4 goals for the IDApps project: Developer oriented, opinionated, portable and self-cleaning.

Developer oriented means you won't find common end-user applications like Calibre, Transmission or Plex in this collection.
Just services and tools that are useful to developers and devops engineers.

Opinionated means that it doesn't try to cover all use cases. In fact, it doesn't try to cover anything but the simplest
use case of these services. When you start postgres you get postgres with a predetermined user and password and that's
it. If you need TLS on the connection, sorry that's not the goal of IDApps. You can modify the script to make it fit
your needs of course, but IDApps aims to target the 70%-80% of use cases that one simply needs a database to hook on
their Spring app as he develops.

Portable means that these shell scripts try hard to limit their requirements to 2. Docker and a POSIX shell. So, no bash,
zsh or other shell-specific commands. As it is, it should run on all Linux (including WSL), BSD and MacOS.

Self-cleaning means that no stuff are left behind after use. All containers are removed when stopped (explicitly or from
a reboot), and the persistent storage (if any) is removed with the `clean` command. The only thing that remains are the
docker images. If they start to pile up you may clean them with `docker image prune`.

## What it is not

**Major warning!** These scripts are not meant to be used in any form of production or even staging deployment. There
are default passwords and unencrypted endpoints everywhere. This is a development tool!

## Project goals

I will continue to add tools to this collection as I need them and improve the overall code base. I plan to keep the apps
fairly updated too. The scripts are pretty simple, so if your favorite service is not there, feel free to copy, paste,
modify add send me a pull request.

## How to install

Just clone the project and add it last to your PATH. See details in the [project page on GitHub](https://github.com/BabisK/idapps). 
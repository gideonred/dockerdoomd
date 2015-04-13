# Docker DOOM
## Kill your running Docker container using Id's DOOM!

Tired of killing Docker containers by having to type in
`docker rm <docker_container>`? With every dull stroke of keys on the keyboard
you only wish that you could be saving earth from a possible invasion from
hell? Worry no longer! Now you can kill those Docker containers with the
proper tool, a rocket launcher (or BFG, or shotgun, or whatever)!

https://youtu.be/E1Lm1NFthX8

## Stop talking, I want to run this now

Download and extract this binary on the Linux machine running docker:

https://gideonred.com/bins/dockerdoomd.tar.gz

Start up a few docker containers e.g.,:

```bash
for i in {1..2} ; do docker run -d -t ubuntu:14.04; done
```

Now run the downloaded docker binary:

```bash
./dockerdoomd
```

You should receive output similar to:

```
╭─gideon@localhost  ~
╰─$ ./dockerdoomd
2015/01/24 16:50:50 Pulling image from public repo
Pulling repository gideonred/dockerdoom
f5abca9b93a3: Download complete
511136ea3c5a: Download complete
53f858aaaf03: Download complete
837339b91538: Download complete
615c102e2290: Download complete
b39b81afc8ca: Download complete
3972ba383c15: Download complete
90c7ac13f81e: Download complete
Status: Downloaded newer image for gideonred/dockerdoom:latest
2015/01/24 16:53:05 Image downloaded
2015/01/24 16:53:05 Trying to start docker container ...
2015/01/24 16:53:05 Waiting 5 seconds for "dockerdoom" to show in "docker ps". You can change this wait with -dockerWait.
PORT=5900
2015/01/24 16:53:10 Docker container started, you can now connect to it with a VNC viewer at port 5900
```

Get a VNC Viewer up and running. You could try [Chicken of the VNC](http://sourceforge.net/projects/cotvnc/).

Connect the VNC Viewer to the machine running `dockerdoomd` at port 5900. The password is `1234`.

![](https://gideonred.com/images/vncdockerdoomd.png)

After a few seconds you will see doom appear:

![](https://gideonred.com/images/vncdockerdoomd2.png)

Now if you want to get the job done quickly enter the cheat `idspispopd` and walk through the wall on your right. You should be greeted by your docker containers as little pink monsters. Press `CTRL` to fire. If the pistol is not your thing, cheat with `idkfa` and press `5` for a nice surprise. Pause the game with `ESC`. Feel free to start up and shutdown docker containers in the background while the game is running.

Sounds familiar? This is based of the work done for psdoom. psdoom was used to kill *nix processes.

## How does this magic work?

![](https://gideonred.com/images/dockerdoommeme.jpg)

There is several parts at play. Let&rsquo;s list them:

* The Docker binary, used to start and query Docker containers.
* The DOOM Docker container, running DOOM inside of it, called `dockerdoom`.
* `dockerdoomd`, a daemon that starts the DOOM Docker container, sets everything up and enables the DOOM Docker container to query and stop Docker containers.
* a socket file, enabling communication between DOOM and `dockerdoomd`
* a VNC tcp connection, enabling a connection between DOOM&rsquo;s X11 session and whatever computer you want it to be displayed on.

The best part of all of this, is that DOOM is running within a container. This allows easier deployment of DOOM to whoever wants to run it. So, not only is this a cool magic trick, but it also uses the awesome features that containers give us.

It&rsquo;ll probably be best to explain this by drawing it all out.

![](https://gideonred.com/images/dockerdoomdiag.png)

When you start `dockerdoomd` on your Linux host it will download the
DOOM docker image from the public repo. It will start this image as a container
named `dockerdoom`. After starting the container it will open a Unix socket between 
itself and the `dockerdoom` container. The container will open an X11 VNC 
session and wait for connections. DOOM will be started when the user first connects to the 
`dockerdoom` container with VNC. DOOM will then periodically poll
the Unix socket for info on the running docker containers on the host. 
`dockerdoomd` will do a `docker ps` execution when it receives a `list` request
on the Unix socket. DOOM will spawn monsters for every docker container it 
reads in response to a `list` request. When you kill a monster in DOOM,
DOOM will send a `kill` request to `dockerdoomd` using the Unix socket.
`dockerdoomd` will then do the corresponding `docker rm`.

You may ask why the need for the `dockerdoomd` daemon and all the Unix socket
communication. A process inside of a Docker container should not be able to talk back to the 
host&rsquo;s docker setup, but the `dockerdoomd` daemon is a way of exposing a subset
of docker commands to a specified docker container.

A Docker container having the ability to execute commands or code on the machine hosting the docker container
is something I call a "sudo" Docker container. Simply put, I&rsquo;m giving a docker container
more privileges than is expected from a contained environment.

## Food for thought

### Is their value in having a "sudo" Docker container in real world production environments?

I think so. When maintaining a machine with many containers or VMs you tend to run some control software
on the host. An annoyance is that the control software normally doesn&rsquo;t
use containers or VM&rsquo;s, which means you can&rsquo;t use the benefits of containers or VM&rsquo;s.

Having your control software talk from a container to a host via a well defined API
can help maintainers of the software understand what dependencies need
to be managed and tested when modifying the software.

Of course there will still be some software running natively on the host, but hopefully they
can be written to be as small as possible acting only as proxies or adapters (e.g. `dockerdoomd` in
this case).

### Running X11 or graphical software within docker containers.

I&rsquo;m definitely not the first to do this. It&rsquo;s popular to run Firefox in a container to
add an extra sand box. I have not packaged or built a game for Linux before, but I&rsquo;m sure there is many
who have run into problems packaging games (or other software) and dealing with the various distro&rsquo;s and their quirks.
Containers and streaming technology (naively VNC) gives one possible solution to this problem.

## Links

* [Github repo for the DOOM used here](https://github.com/GideonRed/dockerdoom)
* [Github repo for `dockerdoomd`](https://github.com/GideonRed/dockerdoomd)

## Thanks

Thanks to [orsonteodoro](https://github.com/orsonteodoro) for still keeping psdoom up to date.
I&rsquo;ve based my changes off of his version.

DOOM and related logos are registered trademarks of id Software.

# docker-tunnel
## Use local docker client with a remote machine using an ssh tunnel.

I needed a way to use my local docker client to manage remote docker machines so that I could use compose or Dockerfiles on my local machine to launch containers on the remote host, however I did not want to open the remote host's docker daemon to tcp sockets or manage tls certificates and I needed the solution to be easily portable to multiple client machines.  Unable to find an existing solution that met these requirements I put together the following simple shell script which uses openssh unix socket tunneling. To use put docker-tunnel in the path or somewhere you can easily reference it and make sure it is executable.

## Usage:

- **docker-tunnel init**: Prepare your client to use the docker socket managed by the script.  Run "docker-tunnel init" to setup an initial symlink to the local docker socket and follow the instructions to set the DOCKER_HOST environment variable to begin using this docker-tunnel managed socket.
- **docker-tunnel connect _\[\<user\>@\]\<host\>_**: Redirect the docker-tunnel managed socket with an ssh tunnel to a remote host by running "docker-tunnel connect \[\<user\>@\]\<host\>" and authenticating with standard ssh authentication.  Proceed with using standard docker clients such as docker, docker-compose, etc. which will now use the ssh tunnel to the remote docker socket.
- **docker-tunnel disconnect**: Disconnect the ssh tunnel and reset the socket to a symlink to the local docker socket.
- **docker-tunnel**: With no args will return the \[\<user\>@\]\<host\> currently connected to if any.

For example here I have a local docker and remote docker that happen to be different versions so I can easily see which I'm connected to:

```bash
[monsonnl@localhost ~]$ echo $DOCKER_HOST

[monsonnl@localhost ~]$ docker-tunnel
[monsonnl@localhost ~]$ docker info | grep "Server Version"
Server Version: 17.06.0-ce
[monsonnl@localhost ~]$
```

We don't have the DOCKER_HOST set and there is no tunnel yet.  Lets init now and set the DOCKER_HOST but not connect yet and confirm we are still local.

```bash
[monsonnl@localhost ~]$ docker-tunnel init
# Set the following environment variable to use the docker-tunnel managed docker socket
export DOCKER_HOST=unix:///home/monsonnl/.docker-tunnel/remote-docker.sock
[monsonnl@localhost ~]$ export DOCKER_HOST=unix:///home/monsonnl/.docker-tunnel/remote-docker.sock
[monsonnl@localhost ~]$ echo $DOCKER_HOST
unix:///home/monsonnl/.docker-tunnel/remote-docker.sock
[monsonnl@localhost ~]$ docker-tunnel
[monsonnl@localhost ~]$ docker info | grep "Server Version"
Server Version: 17.06.0-ce
[monsonnl@localhost ~]$
```

Lets connect now and see what we get.

```bash
[monsonnl@localhost ~]$ docker-tunnel connect remotehost
monsonnl@remotehost's password: 
[monsonnl@localhost ~]$ docker-tunnel
remotehost
[monsonnl@localhost ~]$ docker info | grep "Server Version"
Server Version: 1.13.1
[monsonnl@localhost ~]$
```

Wahoo! We are getting the remote machine's info.  Let's disconnect and confirm we return to local.

```bash
[monsonnl@localhost ~]$ docker-tunnel disconnect
Exit request sent.
[monsonnl@localhost ~]$ docker-tunnel
[monsonnl@localhost ~]$ docker info | grep "Server Version"
Server Version: 17.06.0-ce
[monsonnl@localhost ~]$
```

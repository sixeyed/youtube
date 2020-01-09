# Remote Control with Docker Context

> The video for this Docker Nugget is here: https://youtu.be/YX2BSioWyhI

> The docs for the demos are here: https://is.gd/cGD5Gl

## Intro

- Docker Engine manages containers, hosts API
- Docker CLI sends commands to API
- On same box CLI -> API uses private channel

- Can configure Engine to allow remote access (insecure, mTLS, SSH)
- Can configure CLI to talk to remote Engine
- Docker Context simplifies that

## Diagram

```
+---------------------------------+
|                                 |
|       +-----------------+       |
|       |                 |       |
|       |   Docker CLI    +------------------------------------+
|       |                 |       |                            |
|       +--------+--------+       |      HTTP/                 |
|                |                |      mTLS/                 |
|  Linux socket/ |                |      SSH                   |
|  named pipe    |                |                            |
|                |                |                            |
|                |                |               +-------------------------+
|   +-------------------------+   |               |  +---------v-------+    |
|   |  +---------v-------+    |   |               |  |   Docker API    |    |
|   |  |   Docker API    |    |   |               |  +-----------------+    |
|   |  +-----------------+    |   |               |                         |
|   |                         |   |               |    Docker Engine        |
|   |    Docker Engine        |   |               |                         |
|   |                         |   |               +-------------------------+
|   +-------------------------+   |
|                                 |
|                                 |
+---------------------------------+

```

## Demo 1 - Setup Remote Access without TLS

**Do not do this! This exposes your Docker API on the network with no authentication or encryption**

But if you want to try... Update the Docker Engine config on your server to include this line:

```
# Linux
{
    "hosts":  [
        "tcp://0.0.0.0:2375",
        "fd://"
    ]
}

# OR Windows
{
    "hosts":  [
        "tcp://0.0.0.0:2375",
        "npipe://"
    ]
}
```

> That's in `/etc/docker/daemon.json` in Linux or `C:\ProgramData\docker\config\daemon.json` on Windows

> On Debian you'll need to [update the Docker service config too](./docker-service-update-debian.md)

Restart the Docker Engine and then you can connect to the remote machine:

```
curl http://rock64-01.sixeyed:2375/containers/json

docker --host tcp://rock64-01.sixeyed:2375 container ls
```

But it's easier to set up a context to connect to that remote machine:

```
docker context create rock64-01-insecure --docker "host=tcp://rock64-01.sixeyed:2375"
```

Switching contexts lets you manage different engines:

```
docker version

docker context use rock64-01-insecure

docker version

docker container run -d -p 80:80 --name apache diamol/apache
```

> Browse to http://rock64-01.sixeyed

## Demo 2 - Setup Remote Access with TLS

First create your [CA, client and server certificates](./create-tls-certs.md) and copy to server and client.

_On the server:_

- `docker-ca.pem -> /certs/docker-ca.pem`
- `server-key.pem -> /certs/server-key.pem`
- `server-cert.pem -> /certs/server-cert.pem`

_On the client:_

- `docker-ca.pem -> /docker-certs/ca.pem`
- `key.pem -> /docker-certs/key.pem`
- `cert.pem -> /docker-certs/cert.pem`

Update your Docker Engine config to use port `2376` (by convention), and add the TLS settings:

```
# Linux
{
    "hosts":  [
        "tcp://0.0.0.0:2376",
        "fd://"
    ],
    "tlsverify": true,
    "tlscacert": "/certs/docker-ca.pem",
    "tlskey": "/certs/server-key.pem",
    "tlscert": "/certs/server-cert.pem"
}

# OR Windows
{
    "hosts":  [
        "tcp://0.0.0.0:2376",
        "npipe://"
    ],
    "tlsverify": true,
    "tlscacert": "/certs/docker-ca.pem",
    "tlskey": "/certs/server-key.pem",
    "tlscert": "/certs/server-cert.pem"
}
```

Restart the Docker service and set up a context to connect securely:

```
docker context create rock64-01-tls --docker "host=tcp://rock64-01.sixeyed:2376,ca=/docker-certs/ca.pem,cert=/docker-certs/cert.pem,key=/docker-certs/key.pem"
```

Connect with mTLS:

```
docker context use rock64-01-tls

docker version

docker container logs apache
```

The insecure version doesn't work now because the Engine config requires mTLS:

```
docker context use rock64-01-insecure

docker container ls

```

## Demo 3 - Remote Access with SSH

No need to set anything up on the server, the CLI uses an existing SSH user.

```
docker context create rock64-01-ssh --docker "host=ssh://dietpi@rock64-01.sixeyed"
```

Connect with SSH (you'll be prompted for your SSH password every time; really you should use [SSH keys](https://www.booleanworld.com/set-ssh-keys-linux-unix-server/)):

```
docker context use rock64-01-ssh

docker version

docker container rm -f apache
```

## Demo 4 - Managing & Changing Contexts

Docker context commands:

```
docker context
```

- list
- export & import
- remove

Switching contexts:

```
docker context use pine64-01

docker container ls
```

> `context use` permanently switches contexts, so it stays between sessions

New session:

```
docker container rm -f $(docker container ls -aq)  #wait!

docker container ls
```

** It's much better to leave the context as `default` and switch with the environment variable **

```
docker context use default

docker version

$env:DOCKER_CONTEXT='pine64-01'

# or export DOCKER_CONTEXT='pine64-01' on Linux/Mac

docker container ls
```

New session:

```
docker container ls

$env:DOCKER_CONTEXT='rock64-01'

docker container run -d -P diamol/apache

docker container ls
```

Previous session:

```
docker container ls
```

## Links

- [DietPi](https://www.dietpi.com) - lightweight Linux distro for Arm boards
- [Docker API docs](https://docs.docker.com/engine/api/v1.40/)
- [Docker context docs](https://docs.docker.com/engine/context/working-with-contexts/)
- [Learn Docker in a Month of Lunches](https://is.gd/diamol) - my book :)

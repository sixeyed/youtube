# Configuring the Docker Engine on Debian

You can set the config in `/etc/docker/daemon.json` but some settings clash with the startup command in the Docker service.

The host to listen on is one of those - if it's specified in the config file and the startup command, your Engine won't start.

## You need to remove the host setting in the service command

Open the service config file:

```
# use sudo if you're not root

nano /lib/systemd/system/docker.service
```

Find the line that starts `ExecStart` and remove the `-H` argument:

```
# Before
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

# After
ExecStart=/usr/bin/dockerd --containerd=/run/containerd/containerd.sock
```

## Then you can restart Docker

Save and exit the file, then reload the config:

```
systemctl daemon-reload
```

And restart Docker:

```
service docker restart
```

**You will need to do this every time you update Docker, because the update sets the -H argument in the serice again**

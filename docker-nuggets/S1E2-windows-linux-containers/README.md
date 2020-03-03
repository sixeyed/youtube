# Running Windows and Linux Containers with Docker Desktop

> The video for this Docker Nugget is here: [TODO]

> The docs for the demos are here: [TODO]

## Intro

- Docker Desktop runs Windows container natively
- And manages a Linux VM to run Linux containers
- Run containers simultaneously, but separate Docker Engines

- Route traffic between Linux and Windows containers via host
- Manage Kubernetes apps in Windows container mode
- Configure Kubernetes to route to Windows containers

## Diagram

## Demo 1 - Windows and Linux containers

In Linux container mode:

```
docker version

docker container ls

docker container run -d -p 8080:80 --name nginx nginx:alpine
```

> Browse to Nginbx at http://localhost:8080

Switch to Windows container mode. List containers:

```
docker version

docker container ls
```

No containers, but [refresh Nginx](http://localhost:8080) and it is still up.

> The Linux container is still running - the Docker CLI is now pointing to the Windows Docker Engine, where there are no containers running, but the container is still there in the Linux VM.

Check in Hyper-V Manager:

```
Virtmgmt.msc
```

Run a Windows container:

```
docker container run -d -p 8081:80 --name apache diamol/apache
```

> Browse to Apache at http://localhost:8081

Switch back to Linux container mode. List containers:

```
docker container ls
```

Just Nginx - but refresh Apache and both Linux and Windows containers are still running.

## Hello `host.docker.internal`

Docker Desktop adds an entry to the hosts file with the IP address of the machne running Docker, so containers can access the host machine on `host.docker.internal`:

```
cat C:\Windows\System32\drivers\etc\hosts

Get-NetIPAddress -InterfaceAlias Ethernet -AddressFamily IPv4 | Select IPAddress
```

Any containers which publish ports - Linux or Windows - are available to other containers - Linux or Windows - via the `host.docker.internal` address:

```
docker container exec -it nginx sh

wget -q -O - http://host.docker.internal:8081
```

Switch to Windows containers:

```
docker container exec -it apache cmd

curl http://host.docker.internal:8080
```

## And so to Kubernetes

Many organizations with .NET workloads are moving them to Kubernetes. You can run new .NET Core apps in Linux containers on Kubernetes alongside old .NET Framework apps in Windows containers, with a mixed Linux & Windows cluster.

You can't do that with Docker Desktop because the Kubernetes cluster is Linux-only, but you can approximate it...

We're in Windows container mode, but I'm running Kubernetes in Docker Desktop and the Kubernetes CLI is still pointing to the Linux node in the Docker VM:

```
kubectl get nodes
```

Deploy the [Pi demo app](./kubernetes/web/web.yaml), which is .NET Core so it can run in Linux containers with Kubernetes:

```
kubectl apply -f ./web/
```

Check the app with some Pi calculations and the metrics:

- http://localhost:8088/Pi
- http://localhost:8088/Pi?dp=5000
- http://localhost:8088/metrics

Deploy [Prometheus](./kubernetes/prometheus/prometheus.yaml) which the app uses for monitoring:

```
kubectl apply -f ./prometheus/
```

Check Prometheus at http://localhost:9090 and graph `pi_compute_duration_seconds_count`.

## Add a legacy Windows component

There's also an old WCF API for the Pi app - in a mixed Kubernetes cluster that could run in a pod on a Windows node, but you can't do that on Docker Desktop.

Instead we'll run the WCF app as a standalone Windows container and connect it via the host network to the Kubernetes pods.

Run the old WCF service:

```
docker container run -d -p 8089:80 -p 50505:50505 --name pi sixeyed/pi:wcf-2002
```

Check the app and its metrics:

- http://localhost:8089/PiService.svc/Pi
- http://localhost:8089/PiService.svc/Pi?dp=1800
- http://localhost:50505/metrics

Prometheus is configured to scrape a target called `pi-wcf` but there's no DNS entry for that in the cluster so it's failing - http://localhost:9090/targets.

## Add a Kubernetes Service for the Windows container

Deploy an [external name pointing to `host.docker.internal`](./kubernetes/wcf/wcf-service-externalName.yaml) (clever!)

```
kubectl apply -f ./wcf/wcf-service-externalName.yaml
```

Check [Prometheus targets](http://localhost:9090/targets) again - hmm...

Let's connect to the container and see what's up:

```
kubectl exec -it svc/prometheus sh

nslookup host.docker.internal
```

Ah, Kubernetes has its own DNS server which doesn't provide the `host.docker.internal` name from Docker Desktop. We need a [headless service pointing to our local IP address](./wcf/wcf-service-headless.yaml) instead:

```
kubectl apply -f ./wcf/wcf-service-headless.yaml
```

Check [targets](http://localhost:9090/targets) and [graph](http://localhost:9090) again - metrics for the Linux and Windows containers are there :)

## Summary

There isn't a great developer experience right now for Windows devs running mixed workloads (maybe that will come in a future Docker Desktop release taking advantage of WSL2).

You can run a [full Kubernetes cluster with Windows nodes in VMs](https://blog.sixeyed.com/getting-started-with-kubernetes-on-windows/) but that means managing VMs, which is an extra headache.

If the bulk of your app is .NET Core apps which you can run in Linux on Kubernetes, then you can follow this example and run Windows components in Windows containers, connecting them via the host machine.

## Links

- [Pi sample app](https://github.com/sixeyed/pi) - .NET Framework and .NET Core Pi implementations
- [Networking features in Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/networking/#use-cases-and-workarounds)
- [Kubernetes best practices: mapping external services](https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-mapping-external-services)
- [Learn Docker in a Month of Lunches](https://is.gd/diamol) - my book :)

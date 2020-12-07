---
layout: post
title:  Deprecation of Dockershim
pageId: 2
categories:
    - kubernetes
tags:
    - docker
    - kubernetes
comments: true
last_modified_at: "2020-12-07T15:00:00+00:00"
changelog:
  - date: "2020-12-07T15:00:00+00:00"
    message: Added throughs about docker still being a useful tool for developers.
---
[hn-docker-deprecation]: https://news.ycombinator.com/item?id=25279924
[oci]: https://www.opencontainers.org
[oci-spec]: https://github.com/opencontainers/runtime-spec
[runc]: https://github.com/opencontainers/runc
[libcontainer]: https://github.com/opencontainers/runc/tree/master/libcontainer
[cri]: https://github.com/kubernetes/kubernetes/blob/242a97307b34076d5d8f5bbeb154fa4d97c9ef1d/docs/devel/container-runtime-interface.md
[containerd]: https://github.com/containerd/containerd
[docker-engine-1_11]: https://www.docker.com/blog/docker-engine-1-11-runc/
[k8s-1_15]: https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes
[grpc]: http://www.grpc.io
[kubelet]: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet
[cri-o]: https://github.com/cri-o/cri-o
[eks-cri-support]: https://github.com/aws/containers-roadmap/issues/313
[aks-cri-support]: https://github.com/Azure/AKS/releases/tag/2020-11-16
[gke-cri-support]: https://cloud.google.com/kubernetes-engine/docs/concepts/using-containerd

A couple of days ago I found a news in [Hacker News][hn-docker-deprecation] with the title [Kubernetes is deprecating Docker runtime support][hn-docker-deprecation] that made me think a bit about the impact of this news, both in terms to Kubernetes and for Docker, since Docker popularized the container technology, could this be the end of Docker?

Let's talk a bit about Docker and the container eco-system evolution to understand how we got here.

<!--more-->

## Understanding Docker

Docker is not just a container runtime, it's a entire platform to build, publish and run containers that as a lot of features built on-top like networking, volume management, etc.

Looking at Docker from 10.000 feet, you can find two components, **docker** and **dockerd**. The **docker** client (is a CLI tool) allows you to issue commands to **dockerd** to pull, build or run a image among other things, and the **dockerd** is the Docker daemon process that receives client requests and performs the requested actions.


### Evolution of container runtimes: OCI and CRI


Since it's first release in 2014, docker and the entire container ecosystem have been evolving, and a set of standards emerged from it [Open Container Initiative (OCI)][oci] and [Container Runtime Interface (CRI)][cri].

With [OCI][oci] docker broke down the monolith and separated their code for low level interaction with the Linux kernel that allowed them to run containers to a library called [libcontainer][libcontainer] and a tool called [runc][runc] that became the reference implementation of the [OCI runtime specification][oci-spec].

So in it's core the [OCI runtime specification][oci-spec] specifies the format of a Docker image, called bundle and how the resources from the Linux kernel (cgroups, namespaces) are allocated and how to run a image. There is no network management, not image pulling specified at this level, just how to run containers.

So in December 2015 [Docker Engine 1.11][docker-engine-1_11] got released, this was the first version of Docker supporting [runc][runc] and [containerd][containerd].

[containerd][containerd] is a high level container runtime that is available as a daemon and can manage the complete container lifecycle of its host system: image transfer and storage, container execution and supervision, low-level storage and network attachments, etc.

Underneath [containered][containerd] calls [runc][runc] to create the containers.

<center>
  <figure>
    <img src="/assets/images/deprecation-dockershim-docker-engine.png" alt="python">
    <figcaption>Evolution of the Docker engine Monolith<br /><small>image credit <a href="https://github.com/darshanime/notes/blob/master/kubernetes.org#notes" target="_blank">darshanime/notes</a></small>
    </figcaption>
  </figure>
</center>

The release of [Kubernetes 1.5][k8s-1_15] introduced the [Container Runtime Interface (CRI)][cri], a plugin interface which enables [kubelet][kubelet] to use a wide variety of container runtimes ([containered][containerd], [cri-o][cri-o], etc), without the need to recompile. [CRI][cri] consists of a protocol buffers and [gRPC API][grpc], and libraries, with additional specifications and tools under active development.

## Why the deprecation of Dockershim?

Long story short, Docker engine daemon (dockerd) doesn't support [CRI][cri] meaning that in order for Kubernetes to support Docker to run images they need to use a shim and maintain it at their own cost.

### The Dockershim flow

<center>
  <figure>
    <img src="/assets/images/deprecation-dockershim-dockershim.png" alt="python">
    <figcaption>
      Create container flow via dockershim
      <br />
      <small>
        <a href="https://blog.csdn.net/u011563903/article/details/90743853">image</a> based on <a href="https://blog.csdn.net/u011563903" target="_blank">Frank范 blog</a>
      </small>
    </figcaption>
  </figure>
</center>

When the kubelet wants to create a container the following steps are required:

1. Kubelet calls dockershim through the CRI interface (gRPC) to request the creation of the container. The dockershim is embed directly in the kubelet and acts like a CRI server, and receives the creating request.
2. The dockershim gets the kubelet request and translates it to something that the Docker daemon (dockerd) can understand.
3. As we saw in Docker 1.11, the Docker externalized the container creation to containerd, meaning that dockerd then requests containerd to create the container.
4. One containerd gets the request, it created a process called containerd-shim to let containerd-shim operate the container. This happens because the container process needs a parent process to attach to in order to collect stats, state, etc; and if the containerd process is the parent, if containerd hands or gets upgraded all child container processes on the host will have to exit; containerd and shim are not parent-child processes.
5. The containerd-shim just calls runc a command line tool that handles all the low level container operations and implements the OCI interface standard.
6. After runc starts the container, it will exit directly and the containerd-shim will become the container parent process, responsible for collecting the status of the container process, reporting it to containerd and taking over the of the child process of the container after the process with pid 1 in the container exists, cleaning up and ensuring that there are no zombie processes.

### The containerd via cri-plugin

<center>
  <figure>
    <img src="/assets/images/deprecation-dockershim-containerd-cri-plugin.png" alt="containerd cri plugin">
    <figcaption>
      Create container flow via containerd and cri
    </figcaption>
  </figure>
</center>

When we look at the kubelet flow when using containerd 1.1+ we can see that the list of dependencies and complexity gets reduced:

1. The kubelet calls containerd directly via the CRI interface (gRPC) to request the creation of the container.
2. Before containerd 1.1 there was cri-containerd daemon process that implemented the CRI interface received the requests from the kubelet and forward them containerd. Now containerd implements directly the CRI plugin interface receiving the requests directly, it created a process called containerd-shim to let containerd-shim operate the container. This happens because the container process needs a parent process to attach to in order to collect stats, state, etc; and if the containerd process is the parent, if containerd hands or gets upgraded all child container processes on the host will have to exit; containerd and shim are not parent-child processes.
3. The containerd-shim just calls runc a command line tool that handles all the low level container operations and implements the OCI interface standard.
4. After runc starts the container, it will exit directly and the containerd-shim will become the container parent process, responsible for collecting the status of the container process, reporting it to containerd and taking over the of the child process of the container after the process with pid 1 in the container exists, cleaning up and ensuring that there are no zombie processes.


As you can see maintaining the dockershim brings a lot of pain to the Kubenetes team, they need to manage another component just to keep supporting Docker (tracking deprecations, api changes, etc) and at the end of the day Docker is not special and it should expose a CRI interface just like other container runtimes ([containerd][containerd], [cri-o][cri-o]).


So in order to continue using Docker for Kubernetes deployments in the future, the Docker client or daemon will need to become CRI compliant meanwhile everyone should start planning to transition to a CRI compatible runtime like containerd for example.

Some managed Kubernetes services like [AKS][aks-cri-support] or [GKE][gke-cri-support] already are supporting CRI runtimes others like [EKS][eks-cri-support] are still working supporting it.

Don't forget to check if the container tools that you are using also support CRI runtimes and don't just expect to find Docker.

In a last note this doesn't mean that you for local development should stop using Docker, right now the docker platform in my understanding still gives to developers the best user experience and a huge step to tools that simply make our life easier (eg: docker-compose).

Here are some articles that try to explain a bit more in detail the reasons under the what will this entail for Kubernetes and it's users:
- [Don't Panic: Kubernetes and Docker](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)
- [KEP-1985: Removing dockershim from kubelet](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/1985-remove-dockershim)


# References

- [Draw.io diagram used for this article](/assets/images/deprecation-dockershim.drawio)
- [K8S Runtime CRI OCI contained dockershim understanding](https://blog.csdn.net/u011563903/article/details/90743853)

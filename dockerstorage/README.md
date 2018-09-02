# Docker Storage

## 1: What storage drivers does Docker support?

Docker supports several different storage drivers, using a pluggable architecture. The storage driver controls how images and containers are stored and managed on your Docker host.

## 2: Which storage driver is best for my workload?

There are three high-level factors to consider while you make this decision -

- If multiple storage drivers are supported in your kernel, Docker has a prioritized list of which storage driver to use if no storage driver is explicitly configured, assuming that the prerequisites for that storage driver are met:

If possible, the storage driver with the least amount of configuration is used, such as btrfs or zfs. Each of these relies on the backing filesystem being configured correctly. Otherwise, try to use the storage driver with the best overall performance and stability in the most usual scenarios.

overlay2 is preferred, followed by overlay. Neither of these requires extra configuration. overlay2 is the default choice for Docker CE.

devicemapper is next, but requires direct-lvm for production environments, because loopback-lvm, while zero-configuration, has very poor performance.

The selection order is defined in Docker’s source code. You can see the order by looking at the source code for Docker CE 18.03.
Refer : https://github.com/docker/docker-ce/blob/18.03/components/engine/daemon/graphdriver/driver_linux.go#L50 

```
// List of drivers that should be used in an order
	priority = "btrfs,zfs,overlay2,aufs,overlay,devicemapper,vfs"
```

You can use the branch selector at the top of the file viewer to choose a different branch, if you run a different version of Docker.

- Your choice may be limited by your Docker edition, operating system, and distribution. For instance, aufs is only supported on Ubuntu and Debian, and may require extra packages to be installed, while btrfs is only supported on SLES, which is only supported with Docker EE. See Support storage drivers per Linux distribution.

- Some storage drivers require you to use a specific format for the backing filesystem. If you have external requirements to use a specific backing filesystem, this may limit your choices. See Supported backing filesystems.

- After you have narrowed down which storage drivers you can choose from, your choice are determined by the characteristics of your workload and the level of stability you need. See Other considerations for help making the final decision.

## 3: What are supported storage drivers per Linux distros?

At a high level, the storage drivers you can use is partially determined by the Docker edition you use.

In addition, Docker does not recommend any configuration that requires you to disable security features of your operating system, such as the need to disable selinux if you use the overlay or overlay2 driver on CentOS.

### Docker EE and CS-Engine

For Docker EE and CS-Engine, the definitive resource for which storage drivers are supported is the Product compatibility matrix. To get commercial support from Docker, you must use a supported configuration.

Link: https://success.docker.com/article/compatibility-matrix

### Docker CE

Linux distribution	Recommended storage drivers
Docker CE on Ubuntu	aufs, devicemapper, overlay2 (Ubuntu 14.04.4 or later, 16.04 or later), overlay, zfs, vfs
Docker CE on Debian	aufs, devicemapper, overlay2 (Debian Stretch), overlay, vfs
Docker CE on CentOS	devicemapper, vfs
Docker CE on Fedora	devicemapper, overlay2 (Fedora 26 or later, experimental), overlay (experimental), vfs

When possible, overlay2 is the recommended storage driver. When installing Docker for the first time, overlay2 is used by default. Previously, aufs was used by default when available, but this is no longer the case. If you want to use aufs on new installations going forward, you need to explicitly configure it, and you may need to install extra packages, such as linux-image-extra. See aufs.

When in doubt, the best all-around configuration is to use a modern Linux distribution with a kernel that supports the overlay2 storage driver, and to use Docker volumes for write-heavy workloads instead of relying on writing data to the container’s writable layer.


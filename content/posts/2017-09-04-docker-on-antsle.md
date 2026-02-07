---
title:  "Running docker on antsle"
slug: docker-on-antsle
date:   2017-09-04 10:00:00
draft: false
categories: ["antsle", "docker"]
---
I recently got my hands on an [antsle one pro](https://antsle.com/) to use for my home projects. My initial plan was to run a VM on it that would only run docker containers, but now that they released version [0.5.0](https://antsle.com/company/release-announcements/antsleos-0-5-0/) antsleOS can run docker directly.

I was pleasantly surprised to see that they have pre-configered it with the [ZFS storage driver](https://docs.docker.com/engine/userguide/storagedriver/zfs-driver/), so any container you create will be made from a ZFS snapshot, so it is blindingly fast!

If you want to run docker on antsleOS you need to make some minor modifications. By default docker is only available when you are logged in (as it binds to `fd://var/run/docker.sock`), so if you want to be able to use docker on your laptop and have it communicate with the `dockerd` running on your antsle, you need to edit `/etc/conf.d/docker` and make the docker daemon socket bind to the network interfaces

	DOCKER_OPTS="-H unix:///var/run/docker.sock -H tcp://0.0.0.0:2376"

Note the this will let *anyone* on your local network to access it, but since I'm using it at home behind a firewall and for testing only, it is ok for now. I will set it up with [TLS](https://docs.docker.com/engine/security/https/) later.

You also need to start the docker daemon, as it isn't done by default

    service docker start

And if you want the daemon to always start on reboot, run

	rc-update add docker default

Now you set `DOCKER_HOST` on your laptop, and you the `docker` command will communicate with `dockerd` on your antsle

	export DOCKER_HOST="tcp://myantsle.local"

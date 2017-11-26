---
layout: post
title:  "Upgrading antsle with docker"
date:   2017-11-25 15:00:00
categories: antsle, docker
---
I wrote in a [previous post](http://blog.englund.nu/antsle,/docker/2017/09/09/upgrading-docker-on-antsle.html) how I upgraded docker on my antsle, and I'm now running a few containers directly in antsleOS.

If you do this, you need to stop docker when you [upgrade](https://antsle.com/company/release-announcements/antsleos-0-5-0/) antsleOS, or it will fail as it tries to unmount `/var/lib/docker`. Just make sure to run

    service docker stop

before you start the upgrade. I've reported this to `support@antsle.com` so hopefully it won't happen in the future.

I also noted that the upgrade script doesn't check if `ready-sysres.tar.gz` and `antsleOS-0.5.0.img.gz` already have been downloaded!

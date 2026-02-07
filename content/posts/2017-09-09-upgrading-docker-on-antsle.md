---
title:  "Upgrading docker on antsle"
slug: upgrading-docker-on-antsle
date:   2017-09-09 22:00:00
draft: false
categories: ["antsle", "docker"]
---
I wrote in a [previous post](http://blog.englund.nu/antsle,/docker/2017/09/04/docker-on-antsle.html) how to run docker on antsle. After playing around with it, and trying a `Dockerfile` which uses [multi-stage builds](https://docs.docker.com/engine/userguide/eng-image/multistage-build/), I discovered that the version installed was 17.03 CE.

		myantsle ~ # docker version
		Client:
		 Version:      17.03.1-ce
		 API version:  1.27
		 Go version:   go1.8.3
		 Git commit:   c6d412e
		 Built:        Fri Aug  4 17:19:45 2017
		 OS/Arch:      linux/amd64

		Server:
		 Version:      17.03.1-ce
		 API version:  1.27 (minimum version 1.12)
		 Go version:   go1.8.3
		 Git commit:   c6d412e
		 Built:        Fri Aug  4 17:19:45 2017
		 OS/Arch:      linux/amd64
		 Experimental: false

This version doesn't have multi-stage support, you need at least version 17.05, so I had to see if I could upgrade. Luckily there already is a package with a high enough version:

		myantsle ~ # equery meta app-emulation/docker
		 * app-emulation/docker [gentoo]
		Maintainer:  admwiggin@gmail.com (Tianon)
		Maintainer:  williamh@gentoo.org (William Hubbs)
		Maintainer:  mrueg@gentoo.org (Manuel Rüger)
		Upstream:    Remote-ID:   docker/docker ID: github
		Homepage:    https://dockerproject.org
		Location:    /usr/portage/app-emulation/docker
		Keywords:    17.03.1:0: amd64
		Keywords:    17.03.2:0:
		Keywords:    17.06.0-r1:0:
		Keywords:    17.06.1:0: ~amd64 ~arm
		Keywords:    9999:0:
		License:     Apache-2.0

So all I had to do was to run the upgrade and tell it which version I wanted

		myantsle ~ # emerge -aDuN =app-emulation/docker-17.06.1

And after that run

		myantsle ~ # etc-upgrade

Which got me docker version 17.06 CE

		myantsle ~ # docker version
		Client:
		 Version:      17.06.1-ce
		 API version:  1.30
		 Go version:   go1.8.3
		 Git commit:   874a737
		 Built:        Sat Sep  9 05:09:04 2017
		 OS/Arch:      linux/amd64

		Server:
		 Version:      17.06.1-ce
		 API version:  1.30 (minimum version 1.12)
		 Go version:   go1.8.3
		 Git commit:   874a737
		 Built:        Fri Sep  8 22:08:11 2017
		 OS/Arch:      linux/amd64
		 Experimental: false

Now I can use my multi-stage `Dockerfile` to build my Go projects. Next step is to setup my own [docker registry](https://docs.docker.com/registry/deploying/) so I can host the builds on the antsle.

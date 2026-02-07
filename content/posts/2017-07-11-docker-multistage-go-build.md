---
title:  "Docker multistage go build"
slug: docker-multistage-go-build
date:   2017-07-11 14:00:00
draft: false
categories: ["golang", "docker"]
---
If you ever had to build a go binary and include it in a docker container,
you would either have to do a multi-stage build, which meant you couldn't
use a docker hub automated build, or you had to include the whole build
environment in the image, which meant you significantly bloat it.

In docker `17.06` you can do
[multi-stage builds](https://github.com/moby/moby/pull/32063),
which does away with this issue.

I've created a sample
[github repo](https://github.com/pmenglund/docker-go-mulitistage-build)
to illustrate this,

    # build stage
    FROM golang:alpine AS build
    COPY . /src
    RUN cd /src && go build -o app

    # final stage
    FROM scratch
    WORKDIR /
    COPY --from=build /src /
    CMD ["/app"]

With go binaries, you don't even need a base image, so you can use `scratch`, which saves a few MB.
You can see in the image list that the difference between `go:golang`, `go:alpine` and `go:scatch`,
so doing a multistage build and using `scatch` will save you 257MB.

    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
    go                  scratch             ea0352afec72        4 seconds ago        1.59MB
    go                  alpine              4bba42611772        7 minutes ago        5.55MB
    go                  golang              646c6f10c26b        8 minutes ago        259MB    
    golang              alpine              310e63753884        13 days ago          257MB
    alpine              latest              7328f6f8b418        13 days ago          3.97MB

Unfortunately docker hub runs version `17.03.1-ee-2` at the moment, so the
[automated build](https://hub.docker.com/r/pmenglund/docker-go-mulitistage-build/)
of my github repo
[doesn't work](https://hub.docker.com/r/pmenglund/docker-go-mulitistage-build/builds/bvgsxjrutj6yuomv6phr3vf/),
but once they upgrade it should start building ok.

## Note

Whenever you create a docker image for production use, you should always include a [`HEALTHCHECK`](https://docs.docker.com/engine/reference/builder/#healthcheck).
I've built a very simple `curl`-like go program called
[`gurl`](https://github.com/pmenglund/gurl)
that can be included in the image and used like this

    HEALTHCHECK --interval=5m --timeout=3s CMD /gurl http://localhost:8080/

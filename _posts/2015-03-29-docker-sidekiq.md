---
layout: post
title:  "Running sidekiq in a docker container"
date:   2015-03-29 15:14:00
categories: docker
---
In a project I'm working on we have multiple application components which use [sidekiq](http://sidekiq.org/).
I wanted to run just one docker container with [sidekiq](http://sidekiq.org),
and I was certain I could find information about how to run it in docker using a quick
[google search](https://www.google.com/webhp?hl=en#hl=en&q=docker+sidekiq), but nope!

I've created a [public repository](https://github.com/pmenglund/sidekiq-docker) for this, which you can use
by forking the repo and adding the gems you want to use to the `Gemfile`. This assumes that the workers are
distributed as Ruby gems, which is how we let different components deliver workers to be run by the sidekiq instance.

It relies on [consul](https://www.consul.io/) to find the [redis](http://redis.io/) instance to use, so you should be
running the [progrium/consul](https://github.com/progrium/docker-consul) and
[gliderlabs/registrator](https://github.com/gliderlabs/registrator) docker containers to make life easier for you.

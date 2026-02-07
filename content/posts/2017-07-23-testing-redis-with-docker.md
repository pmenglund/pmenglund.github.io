---
title:  "Testing redis in go with docker"
slug: testing-redis-with-docker
date:   2017-07-23 21:00:00
draft: false
categories: ["golang", "docker", "testing"]
---
If you want to run integration tests in go, and want an easy way to not have to script all the setup and teardown, you can use [`dockertest`](https://github.com/ory/dockertest) helper. It will let you use docker to run services needed for the integration testing, which lets you easily clean up everything after testing is done.

For example if you need to test with [redis](http://redis.io), it will spin up a container just for the duration of the test, and then tear it down once the testing is over.

One worry is that the external dependency will slow you down, but is takes less then 200ms for both setup and teardown on my 2014 MacBook Pro.

I've created created a [github repo](https://github.com/pmenglund/go-testing-redis-with-docker) with [sample code](https://github.com/pmenglund/go-testing-redis-with-docker/blob/master/main_test.go) to show how it can be done.

	$ go test
	PASS
	testing took 181.810519ms
	ok  	github.com/pmenglund/go-testing-redis-with-docker	0.293s

Some things to note with the below code:

* I use a test wrapper to work around the fact that `TestMain` expects to use `os.Exit()`, which ignores `defer`, but this simple wrapper fixes that.
* If you use `docker-machine` you can't use `localhost` for the hostname, as you're connecting to the `docker-machine` image, so you have to get the correct endpoint through the `pool.Client.Endpoint()` call.

Sample test code:

	package main

	import (
		"log"
		"net"
		"net/url"
		"os"
		"testing"

		"github.com/go-redis/redis"
		dockertest "gopkg.in/ory-am/dockertest.v3"
	)

	var client *redis.Client

	func TestMain(m *testing.M) {
		// use a test wrapper, as os.Exit ignores defer, so we can't automatically
		// call `pool.Purge(resource)`
		os.Exit(testWrapper(m))
	}

	func testWrapper(m *testing.M) int {
		pool, err := dockertest.NewPool("")
		if err != nil {
			log.Fatalf("Could not connect to docker: %s", err)
		}

		resource, err := pool.Run("redis", "latest", nil)
		if err != nil {
			log.Fatalf("Could not start resource: %s", err)
		}
		defer pool.Purge(resource)

		// if run with docker-machine the hostname needs to be set
		u, err := url.Parse(pool.Client.Endpoint())
		if err != nil {
			log.Fatalf("Could not parse endpoint: %s", pool.Client.Endpoint())
		}

		if err := pool.Retry(func() error {
			client = redis.NewClient(&redis.Options{
				Addr:     net.JoinHostPort(u.Hostname(), resource.GetPort("6379/tcp")),
				Password: "", // no password set
				DB:       0,  // use default DB
			})

			ping := client.Ping()
			return ping.Err()
		}); err != nil {
			log.Fatalf("Could not connect to docker: %s", err)
		}

		return m.Run()
	}

	func TestGetNext(t *testing.T) {
		expected := getNext(client)
		if expected != 1 {
			t.Errorf("got %d but expected 1", expected)
		}
	}

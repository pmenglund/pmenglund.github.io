---
title:  "Building go programs in jenkins"
slug: golang-in-jenkins
date:   2016-01-17 21:09:00
draft: false
categories: ["jenkins", "go"]
---
I've been working on a go program where the source resides in a private github repo, which means I can't use a build system like [TravisCI](https://travis-ci.org) to build it, so I have to use a private Jenkins instance I'm running.

Getting this to work required some Internet spelunking, so I'm sharing the steps I had to take to get it working here.

First I had to use **Advanced Project Options** and set **Use Custom Workspace** to
`jobs/project/workspace/src/github.com/pmenglund/project`, as it will want to pull down the repo again to `GOPATH` and since this repo is private, it can't be done from the script without sticking the credentials there too (and I don't want to pull it down twice either).

In my **Build** section, my shell script looks like this:

    export GOPATH=/var/lib/jenkins/jobs/project/workspace
    export PATH=$GOPATH/bin:$PATH

    # go get github.com/jstemmer/go-junit-report

    go get
    go test -v ./... | go-junit-report > report.xml

The commented out line is only used once to pull down [go-junit-report](https://github.com/jstemmer/go-junit-report) which I use to with `go test` to be able to output a JUnit report which I use to display the test count on the project page in Jenkins. This is also why I set `GOPATH` and `PATH`, so I don't have to use an absolute path to find `go-junit-report`.

Finally I use **Publish JUnit test result report** and set **Test report XMLs** to `report.xml` so it grabs the test results.

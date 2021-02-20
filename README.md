Docker: the Linux container runtime
===================================

Docker complements LXC with a high-level API which operates at the process level. It runs unix processes with strong guarantees of isolation and repeatability across servers.

Docker is a great building block for automating distributed systems: large-scale web deployments, database clusters, continuous deployment systems, private PaaS, service-oriented architectures, etc.

<img src="http://bricks.argz.com/bricksfiles/lego/07000/7823/012.jpg"/>

* *Heterogeneous payloads*: any combination of binaries, libraries, configuration files, scripts, virtualenvs, jars, gems, tarballs, you name it. No more juggling between domain-specific tools. Docker can deploy and run them all.

* *Any server*: docker can run on any x64 machine with a modern linux kernel - whether it's a laptop, a bare metal server or a VM. This makes it perfect for multi-cloud deployments.

* *Isolation*: docker isolates processes from each other and from the underlying host, using lightweight containers.

* *Repeatability*: because containers are isolated in their own filesystem, they behave the same regardless of where, when, and alongside what they run.


Notable features
-----------------

* Filesystem isolation: each process container runs in a completely separate root filesystem.

* Resource isolation: system resources like cpu and memory can be allocated differently to each process container, using cgroups.

* Network isolation: each process container runs in its own network namespace, with a virtual interface and IP address of its own.

* Copy-on-write: root filesystems are created using copy-on-write, which makes deployment extremeley fast, memory-cheap and disk-cheap.

* Logging: the standard streams (stdout/stderr/stdin) of each process container are collected and logged for real-time or batch retrieval.

* Change management: changes to a container's filesystem can be committed into a new image and re-used to create more containers. No templating or manual configuration required.

* Interactive shell: docker can allocate a pseudo-tty and attach to the standard input of any container, for example to run a throwaway interactive shell.



Under the hood
--------------

Under the hood, Docker is built on the following components:


* The [cgroup](http://blog.dotcloud.com/kernel-secrets-from-the-paas-garage-part-24-c) and [namespacing](http://blog.dotcloud.com/under-the-hood-linux-kernels-on-dotcloud-part) capabilities of the Linux kernel;

* [AUFS](http://aufs.sourceforge.net/aufs.html), a powerful union filesystem with copy-on-write capabilities;

* The [Go](http://golang.org) programming language;

* [lxc](http://lxc.sourceforge.net/), a set of convenience scripts to simplify the creation of linux containers.


Install instructions
==================

Installing on Ubuntu 12.04 and 12.10
------------------------------------

1. Install dependencies:

    ```bash
    sudo apt-get install lxc wget bsdtar curl
    sudo apt-get install linux-image-extra-`uname -r`
    ```

    The `linux-image-extra` package is needed on standard Ubuntu EC2 AMIs in order to install the aufs kernel module.

2. Install the latest docker binary:

    ```bash
    wget http://get.docker.io/builds/$(uname -s)/$(uname -m)/docker-master.tgz
    tar -xf docker-master.tgz
    ```

3. Run your first container!

    ```bash
    cd docker-master
    sudo ./docker run -i -t base /bin/bash
    ```

    Consider adding docker to your `PATH` for simplicity.

Installing on other Linux distributions
---------------------------------------

Right now, the officially supported distributions are:

* Ubuntu 12.04 (precise LTS)
* Ubuntu 12.10 (quantal)

Docker probably works on other distributions featuring a recent kernel, the AUFS patch, and up-to-date lxc. However this has not been tested.

Installing with Vagrant
-----------------------

Currently, Docker can be installed with Vagrant both on your localhost
with VirtualBox as well as on Amazon EC2. Vagrant 1.1 is required for
EC2, but deploying is as simple as:

```bash
$ export AWS_ACCESS_KEY_ID=xxx \
	AWS_SECRET_ACCESS_KEY=xxx \
	AWS_KEYPAIR_NAME=xxx \
	AWS_SSH_PRIVKEY=xxx
$ vagrant plugin install vagrant-aws
$ vagrant up --provider=aws
```

The environment variables are:

* `AWS_ACCESS_KEY_ID` - The API key used to make requests to AWS
* `AWS_SECRET_ACCESS_KEY` - The secret key to make AWS API requests
* `AWS_KEYPAIR_NAME` - The name of the keypair used for this EC2 instance
* `AWS_SSH_PRIVKEY` - The path to the private key for the named keypair

For VirtualBox, you can simply ignore setting any of the environment
variables and omit the `provider` flag. VirtualBox is still supported with
Vagrant &lt;= 1.1:

```bash
$ vagrant up
```



Usage examples
==============

Running an interactive shell
----------------------------

```bash
# Download a base image
docker import base

# Run an interactive shell in the base image,
# allocate a tty, attach stdin and stdout
docker run -i -t base /bin/bash
```


Starting a long-running worker process
--------------------------------------

```bash
# Run docker in daemon mode
(docker -d || echo "Docker daemon already running") &

# Start a very useful long-running process
JOB=$(docker run -d base /bin/sh -c "while true; do echo Hello world; sleep 1; done")

# Collect the output of the job so far
docker logs $JOB

# Kill the job
docker kill $JOB
```


Listing all running containers
------------------------------

```bash
docker ps
```


Expose a service on a TCP port
------------------------------

```bash
# Expose port 4444 of this container, and tell netcat to listen on it
JOB=$(docker run -d -p 4444 base /bin/nc -l -p 4444)

# Which public port is NATed to my container?
PORT=$(docker port $JOB 4444)

# Connect to the public port via the host's public address
echo hello world | nc $(hostname) $PORT

# Verify that the network connection worked
echo "Daemon received: $(docker logs $JOB)"
```

Contributing to Docker
======================

Want to hack on Docker? Awesome! Here are instructions to get you started. They are probably not perfect, please let us know if anything feels wrong or incomplete.

Contribution guidelines
-----------------------

### Pull requests are always welcome

We are always thrilled to receive pull requests, and do our best to process them as fast as possible. Not sure if that typo is worth a pull request? Do it! We will appreciate it.

If your pull request is not accepted on the first try, don't be discouraged! If there's a problem with the implementation, hopefully you received feedback on what to improve.

We're trying very hard to keep Docker lean and focused. We don't want it to do everything for everybody. This means that we might decide against incorporating a new feature.
However, there might be a way to implement that feature *on top of* docker.

### Discuss your design on the mailing list

We recommend discussing your plans [on the mailing list](https://groups.google.com/forum/?fromgroups#!forum/docker-club) before starting to code - especially for more ambitious contributions. This gives other contributors a chance to point
you in the right direction, give feedback on your design, and maybe point out if someone else is working on the same thing.

### Create issues...

Any significant improvement should be documented as [a github issue](https://github.com/dotcloud/docker/issues) before anybody starts working on it.

### ...but check for existing issues first!

Please take a moment to check that an issue doesn't already exist documenting your bug report or improvement proposal.
If it does, it never hurts to add a quick "+1" or "I have this problem too". This will help prioritize the most common problems and requests.


### Write tests

Golang has a great testing suite built in: use it! Take a look at existing tests for inspiration.



Setting up a dev environment
----------------------------

Instructions that have been verified to work on Ubuntu 12.10,

```bash
sudo apt-get -y install lxc wget bsdtar curl golang git

export GOPATH=~/go/
export PATH=$GOPATH/bin:$PATH

mkdir -p $GOPATH/src/github.com/dotcloud
cd $GOPATH/src/github.com/dotcloud
git clone git@github.com:dotcloud/docker.git
cd docker

go get -v github.com/dotcloud/docker/...
go install -v github.com/dotcloud/docker/...
```

Then run the docker daemon,

```bash
sudo $GOPATH/bin/docker -d
```

Run the `go install` command (above) to recompile docker.


What is a Standard Container?
=============================

Docker defines a unit of software delivery called a Standard Container. The goal of a Standard Container is to encapsulate a software component and all its dependencies in
a format that is self-describing and portable, so that any compliant runtime can run it without extra dependencies, regardless of the underlying machine and the contents of the container.

The spec for Standard Containers is currently a work in progress, but it is very straightforward. It mostly defines 1) an image format, 2) a set of standard operations, and 3) an execution environment.

A great analogy for this is the shipping container. Just like Standard Containers are a fundamental unit of software delivery, shipping containers (http://bricks.argz.com/ins/7823-1/12) are a fundamental unit of physical delivery.

### 1. STANDARD OPERATIONS

Just like shipping containers, Standard Containers define a set of STANDARD OPERATIONS. Shipping containers can be lifted, stacked, locked, loaded, unloaded and labelled. Similarly, standard containers can be started, stopped, copied, snapshotted, downloaded, uploaded and tagged.


### 2. CONTENT-AGNOSTIC

Just like shipping containers, Standard Containers are CONTENT-AGNOSTIC: all standard operations have the same effect regardless of the contents. A shipping container will be stacked in exactly the same way whether it contains Vietnamese powder coffee or spare Maserati parts. Similarly, Standard Containers are started or uploaded in the same way whether they contain a postgres database, a php application with its dependencies and application server, or Java build artifacts.


### 3. INFRASTRUCTURE-AGNOSTIC

Both types of containers are INFRASTRUCTURE-AGNOSTIC: they can be transported to thousands of facilities around the world, and manipulated by a wide variety of equipment. A shipping container can be packed in a factory in Ukraine, transported by truck to the nearest routing center, stacked onto a train, loaded into a German boat by an Australian-built crane, stored in a warehouse at a US facility, etc. Similarly, a standard container can be bundled on my laptop, uploaded to S3, downloaded, run and snapshotted by a build server at Equinix in Virginia, uploaded to 10 staging servers in a home-made Openstack cluster, then sent to 30 production instances across 3 EC2 regions.


### 4. DESIGNED FOR AUTOMATION

Because they offer the same standard operations regardless of content and infrastructure, Standard Containers, just like their physical counterpart, are extremely well-suited for automation. In fact, you could say automation is their secret weapon.

Many things that once required time-consuming and error-prone human effort can now be programmed. Before shipping containers, a bag of powder coffee was hauled, dragged, dropped, rolled and stacked by 10 different people in 10 different locations by the time it reached its destination. 1 out of 50 disappeared. 1 out of 20 was damaged. The process was slow, inefficient and cost a fortune - and was entirely different depending on the facility and the type of goods.

Similarly, before Standard Containers, by the time a software component ran in production, it had been individually built, configured, bundled, documented, patched, vendored, templated, tweaked and instrumented by 10 different people on 10 different computers. Builds failed, libraries conflicted, mirrors crashed, post-it notes were lost, logs were misplaced, cluster updates were half-broken. The process was slow, inefficient and cost a fortune - and was entirely different depending on the language and infrastructure provider.


### 5. INDUSTRIAL-GRADE DELIVERY

There are 17 million shipping containers in existence, packed with every physical good imaginable. Every single one of them can be loaded on the same boats, by the same cranes, in the same facilities, and sent anywhere in the World with incredible efficiency. It is embarrassing to think that a 30 ton shipment of coffee can safely travel half-way across the World in *less time* than it takes a software team to deliver its code from one datacenter to another sitting 10 miles away.

With Standard Containers we can put an end to that embarrassment, by making INDUSTRIAL-GRADE DELIVERY of software a reality.




Standard Container Specification
--------------------------------

(TODO)

### Image format


### Standard operations

* Copy
* Run
* Stop
* Wait
* Commit
* Attach standard streams
* List filesystem changes
* ...

### Execution environment

#### Root filesystem

#### Environment variables

#### Process arguments

#### Networking

#### Process namespacing

#### Resource limits

#### Process monitoring

#### Logging

#### Signals

#### Pseudo-terminal allocation

#### Security



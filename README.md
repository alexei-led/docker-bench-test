# Docker Bench Test

[![Circle CI](https://circleci.com/gh/gaia-adm/docker-bench-test.svg?style=svg)](https://circleci.com/gh/gaia-adm/docker-bench-test)  [![](https://badge.imagelayers.io/gaiaadm/docker-bench-test:master.svg)](https://imagelayers.io/?images=gaiaadm/docker-bench-test:master)

The Docker Bench Test is a [Bats](https://github.com/sstephenson/bats) test set that contains tests for dozens of common best-practices around deploying Docker containers in production. The tests are fully automated, are inspired by the [CIS Docker 1.11 Benchmark](https://benchmarks.cisecurity.org/tools2/docker/CIS_Docker_1.11.0_Benchmark_v1.0.0.pdf) and based on [Docker Bench for Security](https://github.com/docker/docker-bench-security).

We are making this available as an open-source utility so the Docker community can have an easy way to self-assess their hosts and docker containers against this benchmark.

## Reasons behind this project

Docker security team released a great open source project: [Docker Bench for Security](https://github.com/docker/docker-bench-security).
With **Docker Bench Test** we have the following goals:

1. Produce a machine readable test results in standard format. Using Bats allows us to produce test results in  [TAP](http://testanything.org/) format.
2. Allow running selected subset of all avaiable tests.
3. Split single container level script into multiple tests. Generate test per container. Thus if one container of tens is not configured properly, only tests for this container will fail.
4. Reduce number of *Info* statuses. *Info* status means that additional manual evaluation is required. Replace manual inspection with predefined "expected" configuration.

## System Requirements

Docker Bench Test requires Docker 1.10.0 or later in order to run.

Also note that the default image and `Dockerfile` uses `FROM: alpine` which doesn't contain `auditctl`, this will generate errors in section 1.8 to 1.18. Distribution specific Dockerfiles that fixes this issue are available in the [distros directory](https://github.com/gaia-adm/docker-bench-test/tree/master/distros).

The [distribution specific Dockerfiles](https://github.com/gaia-adm/docker-bench-test/tree/master/distros) may also help if the distribution you're using haven't yet shipped Docker version 1.10.0 or later.

## Running Docker Bench Bats tests

[Bats](https://github.com/sstephenson/bats) is a [TAP](http://testanything.org/)-compliant testing framework for Bash. It provides a simple way to verify that the UNIX programs you write behave as expected.

**Docker Bench Test*** contains all checks from **Docker Bench for Security** as Bats tests. Container level (and image level) tests are automatically generated for all containers available on host. It's possible to run all or only selected test(s), if you like.

By default TAP test results are reported, but it's possible to produce a "pretty" printed output too.

Use the following command to run Docker Bench Bats tests:

```
Help documentation for docker-bench-test.sh

Basic usage: docker-bench-test.sh [-c] [-p|-t] [-o path] <test> [<test> ...]

Command line switches are optional. The following switches are recognized.
-c  --Displays number of tests. No further functions are performed.
-g  --Generates all CIS Bats tests without execution. No further functions are performed.
-p  --Show results in pretty format.
-t  --Show results in TAP format. This is the default format.
-r  --Create test results files: tests_<timestamp>.tap in test result folder.
-o  --Specify test result folder. Default to /var/docker-bench-test/results.
-h  --Displays this help message. No further functions are performed.

Example: docker-bench-test.sh -t -o /var/docker-bench-test/results
```

**Note:**: You need to run `docker-bench-test.sh` on Docker host as `root` user.

### Running Docker Bench Test tests from Docker image

Then run `docker-bench-tests` container (as bellow). Test results will be saved into `/var/docker-bench-test` folder in TAP format. Test results file is named accoring to the `test_<timestamp>.tap` pattern.

```sh
docker run -it --net host --pid host --cap-add audit_control \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /var/docker-bench-test:/var/docker-bench-test \
    -v /etc/fstab:/etc/fstab \
    -v /etc/docker:/etc/docker \
    -v /etc/default/docker:/etc/default/docker \
    -v /etc/group:/etc/group \
    --label docker_bench_test \
    gaiaadm/docker-bench-test
```

## Build Docker Bench Test Docker image

First, clone and compile your `docker-bench-test` Docker image.

```sh
git clone https://github.com/gaia-adm/docker-bench-test.git
cd docker-bench-test
docker build -t docker-bench-test .
```

or use [Docker Compose](https://docs.docker.com/compose/):
```sh
git clone https://github.com/gaia-adm/docker-bench-test.git
cd docker-bench-test
docker-compose run --rm docker-bench-test
```

Also, this script can also be simply run from your base host by running:

```sh
git clone https://github.com/gaia-adm/docker-bench-test.git
cd docker-bench-test
sh docker-bench-test.sh
```

# Welcome to SalsaBlog
This blog is full of strong opinions loosely held. Evidence supported feedback and criticism is welcome. I'm the kind of crazy that obsesses over software quality, which is why I tend to try out and throw every tool in the book at my disassembler. This blog will document those adventures.

## Code Coverage
Mon Oct 11, 2021

What is code coverage? It's a tool for tracking which lines of code are actually executed during the runtime of a program. In C, this includes how many times each line is hit. Generally, in C, it's presented as a percentage of lines excecuted / lines loaded into memory. Lines loaded into memory is an important distinction. The code coverage percentage can be deceptive because in large projects, some lines and files are ifdef'ed out, or excluded by the build system (therefore not loaded into memory). For many projects, these numbers will be identical or close, but for large projects with mulitple platforms, and architectures, the numbers may diverage dramatically.

One neat trick you can use, when developing in a test driven / code coverage instrumented environment, is dead code elimination.

Some folks track branch coverage, or class coverage but I think focusing on lines is the simplest/biggest bang for your buck. If you're shipping software of any appreciable size and complexity without measuring code coverage for your tests, you're flying blind. It's reckless. Don't do it. A change I hope to help bring in the world is to take software quality and security way more seriously. It's simply not possible to ship high quality secure software without code coverage.

There are endless arguments over how high of a percentage is enough. The real answer, unfortunately, is that 100% is not enough. Let's break this down into necessary and sufficient. It's necessary to measure code coverage to ship high quality secure softwrae, but not sufficient. 

Let's move to the real world where we don't think in absolutes. A lot of projects are anywhere from 0-40% coverage. Focus on getting that up. In my experience adding 



What follows is a practical step by step example of how to measure code coverage in C.

## Not -Werror considered harmful 
Fri Oct 1, 2021

For the majority of my career in the private sector (think closed source), I've taken it for granted that -Wall -Weverything -Werror are always enabled. Projects I found have VC/gcc/clang/Coverity/clang scan-build on day 1, and pull requests to main need a clean build before merging. We also expect 75%+ (ideally 100%) code coverage from automated tests, which also run with the build. So it surprised me to learn that some (many?) folks in the open source and academic world hate -Werror with passion. If we can solve the cons, I think the pros make it self evident -Werror is a good thing.

NB: The examples in this post are extremely simplified. There are about 1,000 tweaks we can do to make this environment more convenient, ie switching easily in and out of the container, make -j, etc.

A non-exhaustive list of other warnings we typically enable:
- -Wtrampolines
- -Wvla
- -Wformat=2
- -Wformat-security


Here are the Pros and Cons as I see them:

### Pros:
1. The codebase is free of bugs that are so obvious a robot with a 1s timeout can find them.
2. We don't ship bugs that simply require a line of code to run to trigger
3. Devs level up, learn and get better
4. The code comes out much tighter. Dead code gets deleted.
5. The code is more portable across OS, processor architectures, and compilers

### (Apparent) Cons:
1. Pull requests can take a significantly longer to merge, especially if they need multiple rounds with the builders
2. You may lose the merge race in fast moving codebases
3. Iteratively developing and building locally with -Werror is super frustrating and unnecessarily time consuming
4. Outsiders or new developers may struggle due to development environment differences

### What can we do about the cons?
#### Con 1. Not a lot except to realize this is actually a Pro.
Left shift is the buzz word that's all the rage right now (usually in presos the following list is a graphic of boxes from left to right, hence left shift). When you consider the cost of a defect, a great first order approximation is that a bug found during each phase of software development costs about 10x the prior phase. This works with time as well, the further along the bug gets the more it costs and the more people it disrupts.

Oversimplified example:
1. At the developers desk $10
2. At the build machine $100
3. After merge during development $1,000
4. QA $10,000
5. Customer $100,000
6. Banner grabbing 0day $1,000,000+

These numbers might sound fantastical but 5 people spending 2 hours who make hundreds of dollars an hour is on the lower end by step 3. More often it's a dozen people independently refinding the bug, re-notifying the owner, and totally DOSed in the mean time. All annoyed and frustrated on top of it.

Meanwhile, in a Github/Bitbucket workflow, this need not actually impose a lot of friction on the poor soul who just wants their code to merge. Take deep breaths. The reality is that the branch awaiting merge is sitting in Bitbucket for anyone else to pull/cherry-pick/rebase on top of locally as needed. Assuming the build works, the artifacts are available for use as well. There's really no downside to holding off that merge until it's ready. It's all upside to disrupt as few people as possible with broken code.

#### Con 2. Again, not a lot
Gotta merge some time. Why should colleagues block on your not quite ready code? Merging conflicts always sucks and is always a problem.

#### Con 3. FTFY
Here again, the Makefiles can save you. Inspired by the V=1 flag from the Linux Makefiles, which enables verbose prints of the compiler command line, we have an I=0 flag. The function of I=0 is to disable -Werror. Why do that, you say? So that when you absentmindedly add an extra unused variable when you're dorking around locally you still get a build. So you make I=0 until just before pushing your branch to open the pull request. No frustrating nuisance warning errors.

Consider the following:
```
ryan$ cat Makefile
CC ?= gcc
CPPFLAGS := $(if $(I),,-Werror) -Wextra -Wall

test.o: test.c Makefile
	$(CC) $(CPPFLAGS) test.c -o test.o

test: test.o
```
```
ryan$ cat test.c
#include <stdio.h>

int main(int argc, char** argv)
{
	int oh_noes_an_unused_variable;
        printf("Hello World!\n");
	return 0;
}

```

I am developing, don't be annoying, compiler:
```
ryan$ make I=0
cc  -Wextra -Wall test.c -o test.o
test.c:5:6: warning: unused variable 'oh_noes_an_unused_variable' [-Wunused-variable]
        int oh_noes_an_unused_variable;
            ^
test.c:3:14: warning: unused parameter 'argc' [-Wunused-parameter]
int main(int argc, char** argv)
             ^
test.c:3:27: warning: unused parameter 'argv' [-Wunused-parameter]
int main(int argc, char** argv)
                          ^
3 warnings generated.

```

I am ready to push my changes for merge, gimme all you got!
```
ryan$ make
cc -Werror -Wextra -Wall test.c -o test.o
test.c:5:6: error: unused variable 'oh_noes_an_unused_variable' [-Werror,-Wunused-variable]
        int oh_noes_an_unused_variable;
            ^
test.c:3:14: error: unused parameter 'argc' [-Werror,-Wunused-parameter]
int main(int argc, char** argv)
             ^
test.c:3:27: error: unused parameter 'argv' [-Werror,-Wunused-parameter]
int main(int argc, char** argv)
                          ^
3 errors generated.
make: *** [test.o] Error 1

```

#### Con 4. Lots!
Having 4 or 5 or even 10 CICD build targets is a tractable problem. Building across current relaeases of all 3 major compilers from day 1 is tractable. Trying to build for every compiler and version is not. The solution is to check a Dockerfile into your project. If you want to get super fancy, and you know I do, have your Makefiles automatically pull and switch you into the build docker. No more struggling with dev environment setup. Install make and the docker client and you're moving.

First a simple Dockerfile. Use this as the input to create a docker image. Be sure to list all your build deps in the Dockerfile.
```
ryan$ cat Dockerfile
FROM ubuntu:latest

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update
RUN apt-get install -y build-essential gdb strace less vim
```

Now create the docker container image. This is done once at project setup, and anytime a new build dependency is added. The docker container ensures everyone, including the build machine, is using an identical build environment. Typically after you create the docker, you push it to a repository so it can be shared with others, and the build machines. I leave the details of that exercise to the reader.
```
ryan$ docker build -t devcontainer:1.0 -f ./Dockerfile .
[+] Building 21.6s (7/7) FINISHED                                                                                                                                                                           
 => [internal] load build definition from Dockerfile                                                                                                                                                   0.0s
 => => transferring dockerfile: 120B                                                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                                       0.5s
 => [1/3] FROM docker.io/library/ubuntu:latest@sha256:44ab2c3b26363823dcb965498ab06abf74a1e6af20a732902250743df0d4172d                                                                                 0.0s
 => CACHED [2/3] RUN apt-get update                                                                                                                                                                    0.0s
 => [3/3] RUN apt-get install -y build-essential gdb strace less vim                                                                                                                                   19.0s
 => exporting to image                                                                                                                                                                                 2.1s 
 => => exporting layers                                                                                                                                                                                2.1s 
 => => writing image sha256:fe8a1e58734c9f0e9b2f6f5ce6fb00ddc62c0e6ad10e36433c814804b0a148ba                                                                                                           0.0s 
 => => naming to docker.io/library/devcontainer:1.0                                                                                                                                                    0.0s 
 ```

Now we can build on our simple Makefile above to automatically pull any new container, if needed, and switch into the container before starting the build.

NB: The docker pull step is commented because setting up a docker repo is out of scope for this post.
```
ryan$ cat Makefile
ifeq ($(DOCKED), 1)

CC := gcc
CPPFLAGS := $(if $(I),,-Werror) -Wextra -Wall

all: test
test.o: test.c | $(MAKEFILE_LIST)
	$(CC) $(CPPFLAGS) -c $? -o $@

test: test.o
	$(CC) -o $@ $?

else
# Prepare to switch into the docker container
VERSION := 1.0
DOCKER_IMAGE := devcontainer:$(VERSION)
DOCKER_ID := -u $(shell id -u):$(shell id -g)

# Share some sensible directories
DOCKER_BIND_MOUNTS := -v $(HOME):$(HOME)
DOCKER_BIND_MOUNTS += -v $(CURDIR):$(CURDIR) -w $(CURDIR)
DOCKER_BIND_MOUNTS += -v /etc/localtime:/etc/localtime
DOCKER_BIND_MOUNTS += -v /etc/passwd:/etc/passwd
DOCKER_BIND_MOUNTS += -v /etc/groups:/etc/groups

# Pass through any important environment
DOCKER_ENV := -e DOCKED=1 -e USERNAME=$(shell whoami)

ifneq ($(I),)
DOCKER_ENV += -e I=$(I)
endif

.NOTPARALLEL:
.PHONY: all
all:
	@echo "Pulling the latest container..."
	# docker pull repo:$(DOCKER_IMAGE)

	# Switch into the docker, grant PTRACE so we an use strace and gdb in the contianer if needed
	@echo "Switching into the docker..."
	docker run --cap-add SYS_PTRACE --rm --log-driver=none --init $(DOCKER_ID) $(DOCKER_BIND_MOUNTS) $(DOCKER_ENV) $(DOCKER_IMAGE) make $(MAKECMDGOALS)

.PHONY: dock
dock:
	@echo "Pulling the latest container..."
	# docker pull repo:$(DOCKER_IMAGE)

	# Switch into the docker, grant PTRACE so we an use strace and gdb in the contianer if needed
	@echo "Switching into the docker..."
	docker run -it --cap-add SYS_PTRACE --rm --log-driver=none --init $(DOCKER_ID) $(DOCKER_BIND_MOUNTS) $(DOCKER_ENV) $(DOCKER_IMAGE) /bin/bash
endif
```

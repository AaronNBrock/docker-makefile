## Introduction
This `Makefile` is used to keep versioning of your Docker images more consistent.  The first and most important thing to talk about is how "Versioning" is to be handled.  The version is generated using [this git command](https://github.com/AaronNBrock/docker-makefile/blob/master/Makefile#L43):

```bash
git describe --always --dirty --match "v[0-9]*"
```

Then, based on the result of that command the docker images are tagged with 3 tags:
1. The project name, for easily testing locally. (ex: `project-name`)
2. The project name with a version. (ex: `project-name:v1.0.0-1-g91f0949`)
3. The project name with either `latest` or `edge`.  `latest` if that commit was tagged, else `edge`. (ex: `project-name:edge`)

And, finally, if your working directory has uncommited changes, your version will have a `-dirty` at the end of it.  And, it will not allow you to push that image to a registry.

That sounds confusing, here are some examples:

* On a commit _not_ tagged with a version, or following a commit tagged with a version.<br>
    **Version**: `1fa88e8`<br>
    **Docker tags**: `project-name`, `project-name:v1.0.0-1-g91f0949`, `project-name:edge`<br>
    **Will push?**  Yes.

* When on a commit tagged with `v1.0.0`.<br>
    **Version**: `v1.0.0`<br>
    **Docker tags**: `project-name`, `project-name:v1.0.0`, `project-name:latest`<br>
    **Will push?**  Yes.

* When on a commit 1 after `v1.0.0`.<br>
    **Version**: `v1.0.0-1-g91f0949`<br>
    **Docker tags**: `project-name`, `project-name:v1.0.0-1-g91f0949`, `project-name:edge`<br>
    **Will push?**  Yes.
 
* When on a commit 1 after `v1.0.0`, but your working directory has uncommited changes..<br>
    **Version**: `v1.0.0-1-g91f0949-dirty`<br>
    **Docker tags**: `project-name`, `project-name:v1.0.0-1-g91f0949-dirty`, `project-name:edge`<br>
    **Will push?**  No.

Pretty neat, huh?


Okay, let's get started!

## Setup

1. Copy `Makefile` into your project, it should be in the same directory as _your_ `Dockerfile`
2. Set `DOCKER_REGISTRY`, `DOCKER_USERNAME` & `PROJECT_NAME` at the top of the Makefile as desired.

3. (optional) Edit recipes under the `RECIPES` header, or add your own.

    For example, if you needed to mount a volume, your `run` might look something like:
    
    ```bash
    # ==== RECIPES ====
    
    run: build
      docker run --rm -v /path/on/host:/path/on/container $(PROJECT_NAME)
    
    run-it: build
	    docker run --rm -v /path/on/host:/path/on/container --entrypoint /bin/sh -it $(PROJECT_NAME) 
    ```
## Usage

#### `make build`
Will build & tag the docker image.

#### `make login`
Attemps to log you into the docker registry.  (See "(Registry Login)[https://github.com/AaronNBrock/docker-makefile#registry-login]")

#### `make push` (depends on `build` & `login`)
If the workding directory isn't dirty, will build, tag then push to the docker registry.

#### `make version`
Prints the version, and what the docker tags would be if built.

#### `make run` (depends on `build`)
Will build, tag, and run the docker image.

#### `make run-it` (depends on `build`)
Will build, tag, and run the docker image in interactive mode.

## Registry Login
This `Makefile` also supports automatically logging in using Environment variables, which can be useful if running this `Makefile` inside a build server, like Jenkins.

To do this, simply pass the username and password in as `USERNAME` and `PASSWORD`.

```bash
USERNAME=user1 PASSWORD=123456 make login
```

(note: You only need to specify the username if it differs from the `DOCKER_USERNAME`.) 







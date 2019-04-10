# Docker-in-Docker Drone plugin

This is a plugin for **Drone 1.0** that is aimed mainly at enabling [Testcontainers](https://www.testcontainers.org) to be used during CI build/test steps.
Due to Drone's architecture, Docker-in-Docker is often the most practical way to run builds that require a Docker daemon.

This plugin:

* Is based upon an Docker-in-Docker image
* Includes a startup script that:
	* Starts a nested docker daemon
	* Optionally starts a pull of required images (in parallel with your build, so as to reduce overall time spent waiting for images to be pulled)
	* Starts a specified build container inside the Docker-in-Docker context, containing your source code and with a docker socket available to it
	* Stores/restores image layers within the CI workspace, for best efforts caching. Even when cached, images will still be pulled to enforce up-to-date checks.

## Prerequisites

* (Drone 1.0): To enable on a per-repository basis, enable the *Trusted* setting for the repository. *Or*

## Usage/Migration (Drone 1.0)

Modify the `build` step of the pipeline to resemble:

```yaml
kind: pipeline
name: default

steps:
  ...
  - name: build
    privileged: true
    image: jopecko/dind-drone-plugin
    settings:
      image: openjdk:8-jdk-alpine
      command:
      - java -version
      # Not mandatory, but enables pre-fetching of images in parallel with the build, so may save time:
      prefetch_images:
      - "redis:4.0.6"
```

When migrating to use this plugin from an ordinary build step, note that:

* Note that _commas_ are not supported within `commands` items due to the way these are passed in between Drone and this plugin.
* `prefetch_images` is optional, but recommended. This specifies a list of images that should be pulled in parallel with your build process, thus saving some time.
* `mounts` is an optional list of `volumes` you can have the dind container mount on your underlying container when its started. Note that they must first be listed in the `volumes` section of this plugin before they can be mounted.

## Copyright

This repository contains code which was mainly developed at [Skyscanner](https://www.skyscanner.net/jobs/), and is licenced under the [Apache 2.0 Licence](LICENSE).

(c) 2017-2019 Skyscanner Ltd.

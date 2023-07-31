---
layout: post
title: Perform unit testing of Qt programs in GitHub Actions using Docker
---

In the [previous post]({% post_url 2022-12-09-build-qt-in-docker-for-github-actions %}), we built a Docker container for Qt and pushed it to [Docker Hub](https://hub.docker.com/r/mbarzegary/qt-5.15.2-freefem-4.10) (the pushed container includes FreeFEM too, but we don't need to care about it for the purpose of this post). Now, it's time to pull the image in a GitHub Action to test the build process and functionality of Qt programs.

We use [this action](https://github.com/addnab/docker-run-action) to pull the image and run the build procedure. First, we add a YAML file, called `main.yml`, to the directory `.github/workflows` in the GitHub repository (like [this example](https://github.com/mbarzegary/BioDeg-UI/blob/main/.github/workflows/main.yml)) and add the following content to it:

```yaml
# This is a basic workflow to help you get started with Actions

name: build

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only 
  # for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run
# sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed 
    # as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, 
      # so your job can access it
      - uses: actions/checkout@v2

      # Pulls a Docker image and builds the software inside it
      - name: Build and test BioDeg
        uses: addnab/docker-run-action@v3
        with:
          registry: docker.io
          image: mbarzegary/qt-5.15.2-freefem-4.10:latest
          options: -v ${{ github.workspace }}:/data 
          run: /data/buildWithDocker.sh
```

This action gets triggered when any commit is pushed to the repository, resulting in the Docker image being pulled and the whole repository being mounted into a directory called `data`. Then, it executes the build script file (in this case, `buildWithDocker.sh` in the root of the repository) inside the container. The following is an example of the `buildWithDocker.sh` file:

```bash
#!/bin/bash
# build the repo inside the docker container
# used for CI builds and tests
cd /data
mkdir build
cd build
cmake ..
make
make test
```

The full running example can be explored in [this repository](https://github.com/mbarzegary/BioDeg-UI). The log of previous ran tests can be viewed [here](https://github.com/mbarzegary/BioDeg-UI/actions/workflows/main.yml).
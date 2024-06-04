# Hello, World! Docker Action

[![GitHub Super-Linter](https://github.com/alissa-huskey/hello-world-docker-action/actions/workflows/linter.yml/badge.svg)](https://github.com/super-linter/super-linter)
![CI](https://github.com/alissa-huskey/hello-world-docker-action/actions/workflows/ci.yml/badge.svg)

> [!IMPORTANT]
>
> This repo was created from a template. Go [here][template] for the real thing.

[template]: https://github.com/actions/hello-world-docker-action

This action prints `Hello, World!` or `Hello, <who-to-greet>!` to the log. To
learn how this action was built, see [Creating a Docker container action][tutorial].

[tutorial]: https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action.

## Use this Template

To create your own action you can use this repository as a template! Just
follow the instructions below:

1. [x] Click the **Use this template** button at the top of the repository.
1. [x] Select **Create a new repository**.
1. [x] Select an owner and name for your new repository.
1. [x] Click **Create repository**.
1. [x] Clone your new repository.
1. [x] Remove or update the [`CODEOWNERS`](./CODEOWNERS) file. ([More info][codeowners].)

[codeowners]: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners.

## :hammer_and_wrench: Create Your Own Action

To make this action work you'll need three files that will tie together the
GitHub action, Docker, and a simple shell script.

- [x] [`action.yml`](#page_facing_up-actionyml)
- [x] [`Dockerfile`](#page_facing_up-dockerfile)
- [x] [`entrypoint.sh`](#page_facing_up-entrypointsh)

### :memo: `action.yml`

The [`action.yml`][] file is where the public GitHub action is defined. In this
case it specifies that the action should run the `Dockerfile` with the
specified inputs and outputs.

This action file specifies the following inputs and outputs:

| Input          | Default | Description                     |
| -------------- | ------- | ------------------------------- |
| `who-to-greet` | `World` | The name of the person to greet |

| Output | Description             |
| ------ | ----------------------- |
| `time` | The time we greeted you |

See the [GitHub Docs on action files][actions-metadata] for more info.

[actions-metadata]: https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions
[`action.yml`]: ./action.yml

### :memo: `Dockerfile`

The [`Dockerfile`][] defines the startup behavior of the Docker container that
will be built and run when your action is triggered. In this case, it
essentially boots up a container and executes the [`entrypoint.sh`][] script.

[`Dockerfile`]: ./Dockerfile

### :memo: `entrypoint.sh`

The [`entrypoint.sh`][] script is executed by the Docker container at runtime.
It does whatever work it needs to in the container, issues commands to the
action, and writes output variables to a a special file referred to by
the environment variable `GITHUB_OUTPUT`.

In this case the script prints a workflow command beginning with `::notice`,
and writes the `time` output variable to the `$GITHUB_OUTPUT` file to be used
by the action later.

[`entrypoint.sh`]: ./entrypoint.sh

## :crossed_fingers: Local Testing

Successfully running an action requires a lot of moving parts that need to work
right individually and also play nice together. Before you can be confident
that your action will run smoothly, you need to make sure all the pieces
function on their own by trying them out locally.

You will need to:

1. [x] Test the script.
1. [x] Build the image.
1. [x] Run the container.
1. [x] Cleanup

> [!TIP]
>
> In the [`bin/`](./bin/) directory there are a few convenience scripts for
> running Docker commands, including [`build`](./bin/build) and
> [`run`](./bin/run) scripts. You should probably type out the commands by hand
> until you get the hang of it though.

### :white_check_mark: Test the script

First try out the [`entrypoint.sh`][] script. You'll need to set the
environment variables `INPUT_WHO_TO_GREET` and `GITHUB_OUTPUT`.

When you run the script, the `::notice` workflow command will be printed, and a
file will be called `output-file` (or whatever you set the `GITHUB_OUTPUT`
environment variable to).

It should look something like this:

```bash
$ INPUT_WHO_TO_GREET="Mona Lisa Octocat" GITHUB_OUTPUT="output-file" ./entrypoint.sh
::notice file=entrypoint.sh,line=7::Hello, Mona Lisa Octocat!
```

The new file should contain the `time` output variable, like this:

```bash
$ cat output-file
time=Mon Jun  3 19:03:50 MDT 2024
```

:sparkles: Congratulations! Your script works.

> [!NOTE]
>
> The environment variable GITHUB_OUTPUT is needed when running the script
> or Docker container locally, but it is provided automatically when the
> the action is run.

### :white_check_mark: Build the image

> [!NOTE]
>
> You'll need to have a reasonably modern version of [Docker][docker] handy
> (e.g. docker engine version 20 or later). 
>
> Also, make sure that the [Docker daemon][daemon] is running!

First you will need to build the image that will be used to create your
container, in this case named `greeting-image`.

```bash
$ docker build --tag greeting-image .
```

You can confirm that the new image was created by running `docker image ls`.

```bash
$ docker image ls
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
greeting-image   latest    0cac2c4c7e42   19 seconds ago   8.83MB
```

:sparkles: Congratulations! Your just built a Docker image.

[docker]: https://www.docker.com/get-started/
[daemon]: https://docs.docker.com/config/daemon/start/

### :white_check_mark: Run the Container

Create and run a new container using the `docker run` command using the image
you just created. The command will look something like this.

```bash
docker run --env NAME="VALUE" --name CONTAINER_NAME IMAGE_NAME
```

You'll need to provide environment variables when you run the container so they
can be passed along to your script.

You can pass them on the command line using the `--env` option.

```bash
$ docker run --env INPUT_WHO_TO_GREET="Mona Lisa Octocat" --env GITHUB_OUTPUT="output-file" --name greeting-container greeting-image
::notice file=entrypoint.sh,line=7::Hello, Mona Lisa Octocat!
```

That can be a little hard to read though, so instead you can save them the
variables to local file named something like `.env` that looks like this:

```bash
INPUT_WHO_TO_GREET="Mona Lisa Octocat"
GITHUB_OUTPUT="output-file"
```

Then use the `--env-file` option to tell docker about it.

```bash
$ docker run --env-file .env --name greeting-container greeting-image
::notice file=entrypoint.sh,line=7::Hello, Mona Lisa Octocat!
```

:sparkles: Congratulations! Your container run worked.

> [!NOTE]
>
> Sometimes these files contain sensitive information, so it is recommended to
> use a hidden filename like `.env` which you don't commit to your repository.
> Add the filename to `.gitignore` to be sure.
>
> You can copy the example [`env`](./env) file, then modify it to suit your needs:
>
>     cp env .env

This container ran, then stopped as soon as the job was done. That means it
won't be shown in the container list by default. If you want to confirm that it
was created though you can use `docker ps --all`.

```bash
$ docker ps --all
CONTAINER ID   IMAGE            COMMAND                    CREATED          STATUS                      PORTS     NAMES
3212c6362f38   greeting-image   "/usr/src/entrypoint..."   23 seconds ago   Exited (0) 22 seconds ago             greeting-container
```

### :white_check_mark: Cleanup

Once you've confirmed everything works, you won't need the image or container
files anymore. You can delete them to free up disk space.

```
$ docker rm greeting-container
$ docker rmi greeting-image
```

Confirm they're gone with `docker image ls` and `docker ps`.

```
$ docker ps --all
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

```
$ docker images --all
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

## :robot: Watch it go!

It's time to give your action a whirl. You'll need to:

* [ ] Create a workflow File.
* [ ] Run the job.

### :memo: Create a Workflow File

In `.github/workflows/` create a new file called `greeting.yml` using the
template below. Be sure to replace `USERNAME` with your GitHub username.

`greeting.yml`:

```yaml
name: Greeting Workflow

on:
  workflow_dispatch:
    inputs:
      who-to-greet:
        description: Who to greet in the log
        required: true
        default: 'World'
        type: string

jobs:
  say-hello:
    name: Say Hello
    runs-on: ubuntu-latest

    steps:
      - name: Print to Log
        id: print-to-log
        uses: USERNAME/hello-world-docker-action@main
        with:
          who-to-greet: ${{ inputs.who-to-greet }}
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

This will create a new workflow with two jobs.

> [!IMPORTANT]
>
> You'll want to change `main` to a to a specific commit SHA or version tag for a
> real public action. For example:
>
> - greeting-docker-action@e76147da8e5c81eaf017dede5645551d4b94427b
> - greeting-docker-action@v1.2.3

### :white_check_mark: Run the Job

Run the workflow from the [Actions tab](actions).

1. [ ] Commit and push your changes.
1. [ ] From your repository, click the **Actions** tab, and select the **"Greeting Workflow"**.
1. [ ] Under **Jobs** or in the visualization graph, click **A job to say hello**.
1. [ ] Click the **Hello world** action step. 
1. [ ] To see the timestamp, click **Get the output time**.

If all goes well you should see you should see` "Hello Mona the Octocat"`
or the name you used for the `who-to-greet` input printed in the log.

:tada: Congratulations, you have finished your GitHub Docker Action! :tada:

Hello, World! Docker Action
===========================

[![GitHub Super-Linter](https://github.com/alissa-huskey/hello-world-docker-action/actions/workflows/linter.yml/badge.svg)](https://github.com/super-linter/super-linter)
![CI](https://github.com/alissa-huskey/hello-world-docker-action/actions/workflows/ci.yml/badge.svg)

> [!IMPORTANT]
>
> This repo was created from a template. Go [here][template] for the real thing.

[template]: https://github.com/actions/hello-world-docker-action

:world_map: Overview
--------------------

This action prints `Hello, World!` or `Hello, <who-to-greet>!` as an
annotation, which can be seen in the job log.

**Follow the official guide on GitHub Docs: [Creating a Docker container action][guide].**

What you'll do here:

- [x] [Use this Template](#genie-use-this-template):
      Generate a repository from this template.
- [x] [Create Your Own Action](#hammer_and_wrench-create-your-own-action):
      The three files you need to make a Docker action.
- [x] [Local Testing](#crossed_fingers-local-testing): Make sure it works.
- [x] [Watch it Go](#robot-watch-it-go): Run your new action.

[guide]: https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action

:genie: Use this Template
-------------------------

To create your own action you can use this repository as a template! Just
follow these instructions.

<details>
<summary>Using GitHub Web</summary>

1. [x] Click the **Use this template** button at the top of the repository.
1. [x] Select **Create a new repository**.
1. [x] Select an owner and name for your new repository.
1. [x] Click **Create repository**.
1. [x] Clone your new repository.
1. [x] Remove or update the [`CODEOWNERS`](./CODEOWNERS) file. ([More info][codeowners].)

</details>

<details>
<summary>Using gh CLI</summary>

1. [x] Clone your new repository.

   ```bash
   gh repo create hello-world-docker-action --template="actions/hello-world-docker-action"
   ```

1. [x] Remove or update the [`CODEOWNERS`](./CODEOWNERS) file. ([More info][codeowners].)

</details>

[codeowners]: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners.

:hammer_and_wrench: Create Your Own Action
------------------------------------------

To make this action work you'll need three files that will tie together the
GitHub action, Docker, and a simple shell script.

- [x] [`action.yml`](#memo-actionyml): GitHub action specification.
- [x] [`Dockerfile`](#memo-dockerfile): Docker container specification.
- [x] [`entrypoint.sh`](#memo-entrypointsh):
      Shell script that runs on the container and communicates with action.

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
action, and writes output variables to a special file referred to by
the environment variable `GITHUB_OUTPUT`.

In this case the script prints a workflow command beginning with `::notice`,
and writes the `time` output variable to the `$GITHUB_OUTPUT` file to be used
by the action later.

See the [GitHub Docs on workflow commands][commands] for more info.

[`entrypoint.sh`]: ./entrypoint.sh
[commands]: https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions

:crossed_fingers: Local Testing
-------------------------------

Successfully running an action requires a lot of moving parts that need to work
right individually and also play nice together.

Before you can be confident that your action will run smoothly, you
need to make sure all the pieces function on their own by trying them
out locally.

1. [x] [Test the script](#white_check_mark-test-the-script): Execute `entrypoint.sh`.
1. [x] [Build the image](#white_check_mark-build-the-image): Create a Docker image.
1. [x] [Run the container](#white_check_mark-run-the-container):
       Create and run a Docker container.
1. [x] [Cleanup](#white_check_mark-cleanup): Delete unneeded files.

> [!TIP]
>
> In the [`bin/`](./bin/) directory there are a few convenience scripts for
> running Docker commands, including [`build`](./bin/build) and
> [`run`](./bin/run) scripts. You should probably type out the commands by hand
> until you get the hang of it though.

### :white_check_mark: Test the script

First try out the [`entrypoint.sh`][] script. You'll need to set the
environment variable `INPUT_WHO_TO_GREET`.

When you run the script, the `::notice` workflow command will be printed, and a
file will created called `output-file`.

It should look something like this:

```bash
$ INPUT_WHO_TO_GREET="Mona Lisa Octocat" ./entrypoint.sh
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
> The environment variable GITHUB_OUTPUT used by the `entrypoint.sh` file is
> provided automatically by the action, but when run locally it defaults to
> `output-file`.

### :white_check_mark: Build the image

> [!NOTE]
>
> You'll need to have a reasonably modern version of [Docker][docker] handy
> (e.g. docker engine version 20 or later).
>
> Also, make sure that the [Docker daemon][daemon] is running.

First you will need to build the image that will be used to create your
container, in this case named `greeting-image`.

```bash
docker build --tag greeting-image .
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
you just created.

```bash
$ docker run --env INPUT_WHO_TO_GREET="Mona Lisa Octocat" --name greeting-container greeting-image
::notice file=entrypoint.sh,line=7::Hello, Mona Lisa Octocat!
```

<details>
<summary>More about environment variables.</summary>

When you provide the `INPUT_WHO_TO_GREET` environment variable to Docker at runtime
it is passed along to `entrypoint.sh`.

In the command above you used the  `--env` option. If you have to pass more than one variable, you can include the `--env` option multiple times.

```bash
docker run --env VAR="VALUE" --env OTHER_VAR="OTHER VALUE" --name CONTAINER_NAME IMAGE_NAME
```

As you can see, that can quickly get out of hand when the variables start to add up.

Instead you can create a file named something like `.docker-env` then use the
`--env-file` option to tell docker about it.

```bash
echo 'INPUT_WHO_TO_GREET="Mona Lisa Octocat"' > .docker-env
```

```bash
$ docker run --env-file .docker-env --name greeting-container greeting-image
::notice file=entrypoint.sh,line=7::Hello, Mona Lisa Octocat!
```

</details>

This container ran then stopped as soon as the job was done. That means it
won't be shown in the docker container list by default. If you want to confirm
that it was created you can use `docker ps --all`.

```bash
$ docker ps --all
CONTAINER ID   IMAGE            COMMAND                    CREATED          STATUS                      PORTS     NAMES
3212c6362f38   greeting-image   "/usr/src/entrypoint..."   23 seconds ago   Exited (0) 22 seconds ago             greeting-container
```

:sparkles: Congratulations! Your container run worked.

### :white_check_mark: Cleanup

Once you've confirmed everything works, you won't need the image or container
files anymore. You can delete them to free up disk space.

```bash
docker rm greeting-container
docker rmi greeting-image
```

Confirm they're gone with `docker image ls` and `docker ps`.

```bash
$ docker ps --all
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

```bash
$ docker image ls --all
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

:robot: Watch it go
-------------------

It's time to give your action a whirl. You'll need to:

- [ ] [Create a workflow File](#memo-create-a-workflow-file):
      The file you need to use your new action.
- [ ] [Run the job](#white_check_mark-run-the-job): 
      Trigger the action and look at the results.

### :memo: Create a Workflow File

A workflow file how people use your action, so you'll need to create one too.

In `.github/workflows/` create a new file called `greeting.yml` using the
template below. Be sure to replace `USERNAME` with your GitHub username.

[`.github/workflows/greeting.yml`][greeting.yml]:

```yaml
name: Greeting Workflow

on:
  workflow_dispatch:
    inputs:
      who-to-greet:
        description: Who to Greet
        required: true
        default: 'World'

jobs:
  say-hello:
    name: Say Hello
    runs-on: ubuntu-latest

    steps:
      - name: Print Greeting
        id: print
        uses: USERNAME/hello-world-docker-action@main
        with:
          who-to-greet: ${{ inputs.who-to-greet }}
      - name: Get the output time
        id: time
        run: 'echo ::notice file=greeting.yml,line=24::The time was: ${{ steps.print.outputs.time }}.'
```

This will create a new workflow "Greeting Workflow" with one job, "Say Hello".

> [!IMPORTANT]
>
> You'll want to change **`main`** to a to a specific commit SHA or version tag
> for a real public action. For example:
>
> - greeting-docker-action@e76147da8e5c81eaf017dede5645551d4b94427b
> - greeting-docker-action@v1.2.3

[greeting.yml]: .github/workflows/greeting.yml

### :white_check_mark: Run the Job

Now you can run your "Say Hello" job.

#### From the Web

You can run the workflow on the GitHub.com web interface from the
[**Actions**](actions) tab.

1. [x] Commit and push your changes.
1. [x] From your repository, click the **Actions** tab, and select the
       **"Greeting Workflow"**.
1. [x] At the top of the list of runs click **Run workflow**. Change the
       name in the text box if you want then click **Run Workflow**.
1. [x] When it starts you will see a new **Greeting Workflow** run at the top
       of the list with a spinning yellow circle. Click it.
1. [x] When the run is complete an **Annotations** section should appear, and
       in it will be your two notices.

    ```text
    Say Hello: entrypoint.sh#L7
    Hello, all the way over there!
    ```

    ```text
    Say Hello: greeting.yml#L24
    The time was: Tue Jun 4 08:00:07 UTC 2024.
    ```

#### Using `gh` CLI

If you use the [`gh`][gh] CLI tool, you can easily run your workflow job from the
command line.

1. [x] Commit and push your changes.
1. [x] Run the workflow:

   ```bash
   $ gh workflow run "Greeting Workflow"
   ✓ Created workflow_dispatch event for greeting.yml at main
   ```

1. [x] Look at the run list and copy the number under `ID`. (In this example
       `9363768000`.)

   ```bash
   $ gh run list --limit 1
   STATUS  NAME               WORKFLOW           BRANCH  EVENT              ID          ELAPSED  AGE
   *       improved workflow  Greeting Workflow  main    workflow_dispatch  9363768000  0s       0m
   ```

1. [x] Watch the run using the `ID` from above. The progress of the job will be
       refreshed at intervals until it completes.

   ```bash
   $ gh run watch 9363768000
   Refreshing run status every 3 seconds. Press Ctrl+C to quit.

   ✓ main Greeting Workflow · 9363768000
   Triggered via workflow_dispatch less than a minute ago

   JOBS
   ✓ Say Hello in 4s (ID 25775263154)
     ✓ Set up job
     ✓ Build USERNAME/hello-world-docker-action@main
     ✓ Print Greeting
     ✓ Get the output time
     ✓ Complete job
   ```

1. [x] Sometimes annotations are displayed by `gh run watch` but it's
       inconsistent from my experience. If you don't see them, use the
       `gh run view` command using the same `ID` from above.

   ```bash
   $ gh run view 9363768000

   ✓ main Greeting Workflow · 9363768000
   Triggered via workflow_dispatch about 13 minutes ago

   JOBS
   ✓ Say Hello in 4s (ID 25775263154)

   ANNOTATIONS
   - Hello, World!
   Say Hello: entrypoint.sh#7

   - The time was: Tue Jun 4 08:26:57 UTC 2024.
   Say Hello: greeting.yml#24


   For more information about the job, try: gh run view --job=25775263154
   View this run on GitHub: https://github.com/USERNAME/hello-world-docker-action/actions/runs/9363768000
   ```

[gh]: https://cli.github.com/

:tada: Congratulations, you have finished your GitHub Docker Action! :tada:

Meta
----

- GitHub: [alissa-huskey/hello-world-docker-action][github]
- Generated from: [actions/hello-world-docker-action][template]
- Guide: [GitHub Docs > Creating a Docker container action][guide]
- License: [MIT](./LICENSE)

[github]: https://github.com/alissa-huskey/hello-world-docker-action

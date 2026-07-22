# Docker Sandboxes

## Default Agents

- Claude
- copilot
- Docker-agent
- Gemini
- opencode
- codex
- cursor
- droid
- kiro
- shell

## Getting Started

- Installing
- Logging In
  - Set Network Policy

## Secrets

Mostly API keys for AI providers. They are proxy managed which means that tools inside the VM cannot read the secret directly. But when using it in a request, they will be replaced by sbx automatically.

Secrets set are immediately available in all running sandboxes.

### GitHub CLI

Assuming you are logged into the CLI on your host machine, your can get a token to use with the command `gh auth token`. This will print the token to standard out.

We can pass this token into secrets one step by echoing the token, and piping it to the set secret command.

```sh
echo "$(gh auth token)" | sbx secret set -g github
```

### API Keys

- global
- per sandbox

## Usage

### Run

At a bare minimum, we can start a sandbox with `sbx run <name of harness>`. For example, `sbx run opencode`.

The name of the sandbox by default will be `harness name`-`directory name`. So if we're in the directory /home/brookzerker/code/my_project and we create a sandbox with open code, then the name of the sandbox will be `opencode-my-project`. Note that the `_` was auto converted into a `-` for the sandbox name.

We can set the name explicitly with the `--name myproject` flag.

#### Reattaching

If we use the same name as a running sandbox, we'll attach to it instead of creating a new one.

```
sbx run --name myproject
```

#### Directories

Directories are mounted with the full path recreated inside the sandbox. This is so that the agent can *see* what we see outside the sandbox. It's mostly to make things easier to debug.

When creating a sandbox, we can attach additional directories that the harness can see by passing them in after the name of the harness.

```sh
sbx run opencode . /home/brookzerker/code/other_project
```

By default, these directories are read/write, but can be marked read only with a `:ro` at the end of the path.

#### Multiple Harnesses in the Same Project

It's possible to create more than one sandbox for each project, each with their own harnesses. When using the `--name` flag, we can even have the same harness across multiple sandboxes in the same repo.

When reconnecting to a sandbox, we can state which harness we want to connect to by using it in the run command.

```sh
sbx run opencode --name my-project
```

#### Flags

##### Clone Mode

If we pass in `--clone` the repo for the project will be cloned into the workspace instead of the directory being read/write. That way it can't make changes without us manually merging them in.

##### Name

Customize the name so it's something that makes sense to use, for example `sbx run opencode --name my-project`

### List

Lists all sandboxes

```sh
docker_sandboxes 
❯ sbx ls
SANDBOX                     AGENT      STATUS    PORTS   WORKSPACE
opencode-docker-sandboxes   opencode   running           /Users/brooks/code/brooks-builds/docker_sandboxes
```

### Stop

We can stop the sandbox manually with the command `sbx stop <sandbox name>`. However, the sandbox will auto stop when the harness exits. For example, pressing <ctrl-c> while using it.

### Remove

Deletes a sandbox, freeing up any space that it was using.

#### Flags

##### All

We can use the `--all` flag to remove all sandboxes at one time.

### Exec

We can run a one-off command inside a sandbox with the `sbx exec <command>`.

This is the same as an exec command in Docker, so we can treat it the same.

## Autocompletions

We can use the `sbx completion` command to generate an autocompletion script that can be used in our `.*rc` files to tab autocomplete sbx commands.

## Copy

We can copy files and folders to/from the sandbox/host machines.

## Execute Commands

We can use the `sbx exec` command to run a command inside the VM. This can be a one-off, or long running command if we use the `-d` to run it in the background.

## Resetting

We can reset Docker sandboxes to a freshly installed state with the `sbx reset` command. This will be a true reset, which means that all instances are deleted, secrets are removed, and even you will be logged out from sbx.

## Network Policy

When first installing Docker Sandboxes, we choose a default network policy like balanced. If we ever need to reset and start over from scratch, we can use the command `sbx policy reset`. This will stop everything, delete all policies, and give us the choice for what presets to use again.

## Interactive Mode

Run sbx without any options or commands and we'll get interactive mode

### Network Log

We can see logs for each request, and if it was allowed or denied. We can also allow or block future traffic based on these rules.

### Network Rules

We can add network rules, both for blocked hosts, and allowed hosts.

### Filesystem Rules

We can see what directory mounts into the sandbox exists

## Copying Files Between Sandbox and Host

We can use the `sbx cp` command to copy files from/to the sandbox.

Differentiate between the host and sandbox based on if the name of the sandbox comes first. This is the same as `scp`.

```sh
sbx cp ./README.md my-sandbox:/tmp/files/
```

As of version `v0.34.0` we cannot copy files between sandboxes, just from a sandbox and our host.

## Ports

We can use `sbx ports --publish host_port:sandbox_port` to add a connection between our host system and the sandbox.

We can use `sbx ports --unpublish host_port:sandbox_port` to remove a connection.

There are more options, but this should solve most of our needs.

### Host Ports

The sandbox can access ports on the host system through the address `host.docker.internal`. This way we can run a local AI server like `Llamma.cpp` or `mcx` and have the sandboxes use it.

To allow the sandbox to see the port, we need to add it to the network policy.

```sh
sbx policy allow network localhost:11434
```

### Inspect Sandboxes

We can inspect sandboxes to see details and debug anything with the `sbx inspect <name>` command.

```sh
docker_sandboxes on  main [!]
❯ sbx inspect shell-docker-sandboxes

  Name:           shell-docker-sandboxes
  Agent:          shell
  Kits:           none
  State:          stopped
  Image:          docker/sandbox-templates:shell-docker
  Image digest:   sha256:39cf20eca861ec92747487af6197f6d916f774bdb98245d267dbd8dfd3debb05
  Auth mode:      not configured
  Workspace:      /Users/brooks/code/brooks-builds/docker_sandboxes
  Network Policy: global (local policy)
  Mount Policy:   allowed
  Proxy:          172.17.0.0:3128
  Secrets:        SANDBOX_STORED_CREDENTIAL___OPENROUTER (env), github (uploaded)
  Ports:          none published
  Sessions:       0

  Daemon:         v0.35.0   uptime 23m

docker_sandboxes on  main [!]
```

## Agents

If we want to run our own custom agents, either self-coded or something not on the list like Hermes, then we have to start with the shell, and install it manually

## CLI Tools

### GitHub CLI

The GitHub CLI tool `gh` is installed by default in all the agents images. However, it's not logged in even if you have the tool logged in on the host. You'll need a key, and to set a secret globally or for each sandbox that you want to have access to `gh`.

## Telemetry

By default, the CLI will collect some basic telemetry, this is opt out, and can be done by setting an environment variable before running any SBX commands.

```sh
export SBX_NO_TELEMETRY=1
```

## Objectives

By the end of this course, you will be able to

- [x] Introduction
  - [x] What is the problem with running AI harnesses in containers
    - If the harness is writing code, we have to treat that code as unknown.
    - Containers aren't safe to run unknown code
  - [x] Two solutions
    - Complete isolation (run your harness on a different computer)
    - Virtual machines (best of both worlds)
  - [x] Describe What Docker Sandboxes Are
    - Program wrapping around hypervisor technology to manage virtual machines as easily as Docker did for containers
  - [x] Describe Why Running AI Harnesses in Sandboxes Is a Good Idea.
    - Very fast to launch and get started with
    - Easy to introduce teams and others to
  - [x] Notes
    - Docker sandboxes has a more compelling use-case for Macs since spinning up VM's on Apple devices are not as easy as Windows and Linux
- [ ] Installing
  - [ ] Install Docker sandboxes on a Mac
  - [ ] install Docker sandboxes on Linux
  - [ ] install Docker sandboxes on Windows
  - [ ] Choose a network policy that works for your
  - [ ] Set up autocompletions in your Linux/mac shell
  - [ ] Set up autocompletions in your Windows shell
  - [ ] Reset sandboxes to factory default with the `sbx reset` command
  - [ ] Opt out of sandbox telemetry
- [ ] Creating a sandbox
  - [ ] Create a sandbox for your harness of choice
  - [ ] Create a sandbox with multiple directories mounted
  - [ ] Create a sandbox with multiple directories mounted with some read only
  - [ ] Create a shell sandbox to set up your custom harness
  - [ ] Create a sandbox with a custom name
- [ ] Secrets
  - [ ] Describe why we want to use the secrets command
  - [ ] set a global secret
  - [ ] set a secret for a specific sandbox
  - [ ] delete a global secret
  - [ ] Delete a secret for a specific sandbox
- [ ] Ports
  - [ ] Expose ports to/from a sandbox
  - [ ] Remove exposed ports to/from a sandbox
  - [ ] Expose ports on the host to the sandbox
- [ ] Sandbox management
  - [ ] Create a sandbox with the create command
  - [ ] start a stopped sandbox
  - [ ] connect to a sandbox
  - [ ] delete a sandbox
  - [ ] Delete all sandboxes at the same time
  - [ ] list all sandboxes
  - [ ] TUI
    - [ ] launch the TUI
    - [ ] inspect sandboxes
    - [ ] allow denied network traffic
    - [ ] deny allowed network traffic
    - [ ] connect to a sandbox
    - [ ] Use exec to run a command on the sandbox
    - [ ] Use exec to get an interactive shell on the sandbox
  - [ ] Copy a file to a sandbox from your host
  - [ ] Copy a file from a sandbox to your host
  - [ ] Inspect sandboxes with the `inspect` command
- [ ] Network
  - [ ] view the network rules
  - [ ] add a network globally
  - [ ] Add a network for a specific sandbox
  - [ ] remove a network globally
  - [ ] Remove a network for a specific sandbox
- [ ] Git Management
  - [ ] Describe what the harness can do in a git repo by default
  - [ ] Use the `--clone` flag to clone the repo into the sandbox
  - [ ] Pull in changes from a cloned sandbox
  - [ ] Use a work tree in a sandbox

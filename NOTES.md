# Notes

## Step 0: install + first containers (2026-10-14)

### Setup
- Docker Engine 29.9.0 + Compose v5.6.0, installed inside Ubuntu 26.04 WSL from Docker's own apt repo
  (`/etc/apt/sources.list.d/docker.sources`, suite `resolute`), not Docker Desktop. Reason: the laptop has
  6.9 GB RAM and WSL is capped at 5 GB, so skipping Desktop's extra memory use matters.
- The daemon is a systemd service: `systemctl status docker`. It starts with Ubuntu.
- My user is in the `docker` group, so no `sudo docker`. **That group is root-equivalent**: anyone who can
  run `docker` can mount `/` into a container and edit anything.
- Terminal tabs that were already open before the install don't have the new group yet. Close and reopen them.
- Keep Docker Desktop's WSL integration **off** for Ubuntu, or two Dockers fight over the `docker` command.
- Work in `~/...` (the Linux disk), not `/mnt/g/...`. Windows drives go through a slow file bridge.

### Exercise 0: `docker version`
It prints two halves, **Client** and **Server**. `docker` is only a CLI. It sends requests over
`/var/run/docker.sock` to the daemon `dockerd`, which does the real work (pulls images, creates containers).

### Exercise 1: image vs container
Ran `docker run hello-world` twice.
- 1st run: `Unable to find image 'hello-world:latest' locally` → pulled from Docker Hub. 2nd run: no pull, cached.
- `docker images` → **1** image. `docker ps -a` → **2** containers, both `Exited (0)`.
- **Image** = read-only template (a packaged filesystem + the command to run). **Container** = one run
  of it, with its own writable layer. Like a class and its objects.
- Stopped containers stick around until removed. `docker run --rm ...` deletes it on exit.

### Exercise 2: what a container actually is

| | `uname -r` (kernel) | `/etc/os-release` (OS files) | hostname | processes |
|---|---|---|---|---|
| My Ubuntu (WSL) | 6.6.114.1-microsoft-standard-WSL2 | Ubuntu 26.04 LTS | Zel | 42 |
| `ubuntu` container | 6.6.114.1-microsoft-standard-WSL2 | Ubuntu 26.04.1 LTS | 4c14eaa7a8f6 | 1 (`ps aux` itself, PID 1) |
| `alpine` container | 6.6.114.1-microsoft-standard-WSL2 | Alpine Linux v3.24 (no `/bin/bash`!) | (random) | |

**Why the kernel matched but the OS didn't:** an image has no kernel. It only contains files
(`/bin`, libraries, `/etc/os-release`). A container is an ordinary process on the host's kernel that is
shown a different filesystem. So every container reports the same WSL2 kernel, but each one sees its
own distro's files. That's why Alpine has no bash: its files simply don't include it.

**Why `ps aux` showed almost nothing:** the container gets its own *PID namespace*. It sees only its
own processes, and its first one becomes PID 1. The random hostname is the same trick for the hostname
(*UTS namespace*). Namespaces control what a container can *see*. *cgroups* control how much CPU and RAM
it can *use* (Step 4).

**The games question, answered:** an image brings files, not a kernel. Life is Strange needs the Windows
kernel and DirectX, and a Linux container only ever gets a Linux kernel. Docker on Windows needs a Linux
VM (WSL2) just to have a Linux kernel at all. You can see it in the kernel name: `microsoft-standard-WSL2`.
A container is not a VM. It's an isolated process, and that's also why containers start in milliseconds.

### Exercise 3: cleanup
`docker container prune -f` deleted the 2 stopped hello-world containers. The images stay:
`hello-world` 25.9 kB, `alpine` 13 MB, `ubuntu` 162 MB. Note how small Alpine is (Step 2: base image choice).

## Cheat sheet

| Command | Does |
|---|---|
| `docker run IMAGE` | create + start a container (pulls the image if missing) |
| `docker run --rm -it IMAGE bash` | interactive shell, auto-delete on exit |
| `docker ps` / `docker ps -a` | running / all containers |
| `docker images` | local images |
| `docker logs NAME` / `-f` | container output / follow it |
| `docker exec -it NAME bash` | shell inside a running container |
| `docker stop NAME` / `docker rm NAME` | stop / delete a container |
| `docker container prune` | delete all stopped containers |
| `docker image prune -a` | delete unused images |
| `docker system df` | disk used by images, containers, volumes |

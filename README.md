# Rocky Linux Container Lab

A lightweight Rocky Linux 10 lab with systemd support using Docker or Podman, enabling hands-on enterprise Linux administration without the overhead of virtual machines.

## Introduction

Not everyone has access to a Red Hat Enterprise Linux (RHEL) subscription or the resources to run a virtual machine. This repository provides commands to start a Rocky Linux 10 environment using containers.

No Dockerfiles or automation scripts are required. Run the commands yourself and learn what each one does.

## Requirements

- A Linux host with Docker or Podman installed
- Internet access
- A Linux host with cgroup support for running systemd inside containers

These instructions are intended for native Linux. Container behavior may differ on Docker Desktop or Podman machines.

## 1. Pull the Rocky Linux Image

Choose either Docker or Podman and follow the corresponding commands throughout this guide.

**Docker:**

```bash
docker pull rockylinux/rockylinux:10-ubi-init
```

**Podman:**

```bash
podman pull rockylinux/rockylinux:10-ubi-init
```

**Explanation:**

- `docker` or `podman` is the container management tool.
- `pull` downloads the image.
- `rockylinux/rockylinux` is the Rocky Linux image repository.
- `10-ubi-init` selects the Rocky Linux 10 UBI init image.

## 2. Run Rocky Linux Using Docker

```bash
docker run -d \
  --name rocky10 \
  --privileged \
  --network host \
  rockylinux/rockylinux:10-ubi-init
```

**Explanation:**

- `docker run` creates and starts a container.
- `-d` runs the container in the background.
- `--name rocky10` names the container `rocky10`.
- `--privileged` grants extensive host-level permissions to the container. Use only on a trusted lab host.
- `--network host` shares the host network namespace on native Linux.
- `rockylinux/rockylinux:10-ubi-init` specifies the image.
- `\` continues the command onto the next line.

Enter the container:

```bash
docker exec -it rocky10 bash
```

- `exec` runs a command inside a running container.
- `-i` keeps standard input open.
- `-t` allocates a terminal.
- `rocky10` is the container name.
- `bash` starts a shell.

**Note:** This is a minimal Docker command. Systemd needs appropriate cgroup access to manage services. On some hosts, additional cgroup configuration may be required even when using `--privileged`.

## 3. Run Rocky Linux Using Podman

Podman can run containers as a regular user, without `sudo`.

```bash
podman run -d \
  --name rocky10 \
  --network host \
  rockylinux/rockylinux:10-ubi-init
```

**Explanation:**

- `podman run` creates and starts a container.
- `-d` runs it in the background.
- `--name rocky10` names the container `rocky10`.
- `--network host` shares the host network namespace on native Linux.
- `rockylinux/rockylinux:10-ubi-init` specifies the image.
- `\` continues the command onto the next line.

Podman normally detects systemd automatically in compatible init images. Rootless systemd functionality still depends on host cgroup delegation.

Enter the container:

```bash
podman exec -it rocky10 bash
```

- `exec` runs a command inside a running container.
- `-i` keeps standard input open.
- `-t` allocates a terminal.
- `rocky10` is the container name.
- `bash` starts a shell.

## 4. Check systemd Status

Run this **inside** the container:

```bash
systemctl status
```

**Explanation:**

- `systemctl` manages and inspects systemd.
- `status` shows the system manager's status.

This checks whether the systemd manager responds. Individual services may still have limitations depending on the container runtime and host configuration.

## 5. Host Networking

Both run commands use `--network host`. On native Linux, the container shares the host's network namespace, so you do not need to publish individual ports with `-p` (the port-publishing option).

For example, if an HTTP application is running inside the container on port 8080, run this **on the host**:

```bash
curl http://localhost:8080
```

- `curl` requests the specified URL.
- `localhost` refers to the host's loopback interface.
- `8080` is the port used by the example application.

The application must already be running and listening on the appropriate address and port. Host and container services can conflict when they try to use the same address and port. Services listening on external interfaces may be accessible from other machines. Rootless Podman generally cannot bind ports below 1024 without additional host configuration.

## 6. Manage the Container

If you are still inside the container shell, return to the host first:

```bash
exit
```

- `exit` closes the current shell without stopping the container.

Run the following commands **on the host**.

### List running containers

Docker:

```bash
docker ps
```

Podman:

```bash
podman ps
```

- `ps` lists running containers.

### Stop the container

Docker:

```bash
docker stop rocky10
```

Podman:

```bash
podman stop rocky10
```

- `stop` requests a graceful shutdown.
- `rocky10` is the container name.

### Start the container again

Docker:

```bash
docker start rocky10
```

Podman:

```bash
podman start rocky10
```

- `start` starts an existing stopped container.

### Restart the container

Docker:

```bash
docker restart rocky10
```

Podman:

```bash
podman restart rocky10
```

- `restart` stops and starts an existing container.

### Remove the container

Stop the container first, then run:

Docker:

```bash
docker rm rocky10
```

Podman:

```bash
podman rm rocky10
```

- `rm` removes a stopped container.

Changes made inside a container survive a stop/start cycle but are generally lost when the container is removed unless stored separately.

## 7. Troubleshooting

If the container exits or systemd does not work, inspect its logs.

Docker:

```bash
docker logs rocky10
```

Podman:

```bash
podman logs rocky10
```

- `logs` displays recorded container output.

To include stopped containers in the listing:

Docker:

```bash
docker ps -a
```

Podman:

```bash
podman ps -a
```

- `-a` includes stopped containers.

Systemd requires appropriate cgroup access. The minimal commands above may need adjustments on some hosts.

## Official Links

- [Rocky Linux](https://rockylinux.org/)
- [Rocky Linux Documentation](https://docs.rockylinux.org/)
- [Rocky Linux Container Images](https://hub.docker.com/r/rockylinux/rockylinux)
- [Docker Documentation](https://docs.docker.com/)
- [Podman Documentation](https://docs.podman.io/)
- [systemd Documentation](https://systemd.io/)

## Disclaimer

This project is for learning and testing. Containers share the host kernel and are not full virtual machines. The Docker example uses privileged mode, and both examples use host networking, which reduces isolation. Use them only on trusted systems.

## License

Licensed under the MIT License. See [LICENSE](LICENSE) for details.

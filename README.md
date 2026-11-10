# qbittorrent (fork)

A Docker image running [qBittorrent](https://www.qbittorrent.org/) (`qbittorrent-nox`, the headless client with its web UI), with [qbittorrent-cli](https://github.com/fedarovich/qbittorrent-cli), `unrar` and 7-Zip.

This is a fork of [linuxserver/docker-qbittorrent](https://github.com/linuxserver/docker-qbittorrent), maintained at [PabloVTM/docker-qbittorrent](https://github.com/PabloVTM/docker-qbittorrent). The base image, s6 init and the startup scripts come from upstream; qBittorrent itself is the static build from [userdocs/qbittorrent-nox-static](https://github.com/userdocs/qbittorrent-nox-static).

## Image

Images are published to GitHub Container Registry for `linux/amd64` and `linux/arm64`:

| Tag | Content |
| --- | --- |
| `ghcr.io/pablovtm/qbittorrent:latest` | Latest build of `master`. |
| `ghcr.io/pablovtm/qbittorrent:YYYY-MM-DD` | Dated version, e.g. `2026-11-10`. Each build takes the latest stable qBittorrent and qbittorrent-cli releases and upgrades the Alpine packages, so a new dated version also brings the latest security fixes. |

To build locally instead of pulling the published image, replace `image:` with `build: .` below.

## Quick start

```yaml
---
services:
  qbittorrent:
    image: ghcr.io/pablovtm/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - ./config:/config
      - /path/to/downloads:/downloads #optional
    ports:
      - 8080:8080
      - 6881:6881
      - 6881:6881/udp
    stop_grace_period: "10s" #optional
    restart: unless-stopped
```

```bash
docker compose up -d
```

Then open http://localhost:8080 and sign in as `admin` with the temporary password printed in the log (`docker logs qbittorrent`). Set your own password in **Options → WebUI**: until you do, a new temporary password is generated at every start.

| Setting | Purpose |
| --- | --- |
| `PUID`, `PGID` | User and group that own `/config` and the downloads (`id your_user` gives them). |
| `TZ` | Time zone, e.g. `Europe/Paris`. |
| `WEBUI_PORT` | Port of the web UI. Because of qBittorrent's CSRF checks, change both sides of the port mapping and this variable together (`-p 8123:8123 -e WEBUI_PORT=8123`). |
| `TORRENTING_PORT` | Incoming peer port, TCP and UDP. Same rule: map the same port on both sides (`-p 6887:6887 -p 6887:6887/udp -e TORRENTING_PORT=6887`). |
| `/config` | qBittorrent settings, state and logs. |
| `/downloads` | Download location (optional). |
| `stop_grace_period` | Raise it with many torrents, so qBittorrent can save its state on shutdown. |

The image can also run with a read-only filesystem or as a non-root user (`user: 1000:1000`), as described in the LinuxServer.io docs for [read-only](https://docs.linuxserver.io/misc/read-only/) and [non-root](https://docs.linuxserver.io/misc/non-root/) operation.

## Security

> [!WARNING]
> Do not expose the web UI to the Internet as is. Put it behind a reverse proxy with TLS and authentication, or a VPN, and set a strong password first.

## Maintenance

```bash
# Shell in the container
docker exec -it qbittorrent /bin/bash

# Follow logs
docker logs -f qbittorrent

# qBittorrent version
docker exec qbittorrent /app/qbittorrent-nox --version
```

To update, pull the new image (`docker compose pull`) and recreate the container. Your `/config` volume is kept.

## Releasing

Images are built by GitHub Actions. A push to `master` publishes `latest`; a dated version is published by pushing a tag named after the day:

```bash
git tag 2026-11-10
git push origin 2026-11-10
```

The workflow builds both architectures and checks their binaries, starts the container, signs in to the Web API with the temporary password, adds a torrent and restarts it, then publishes `ghcr.io/pablovtm/qbittorrent:2026-11-10` and `latest` for `linux/amd64` and `linux/arm64`. There is no scheduled rebuild: tag a new version when you want one.

## Building

```bash
docker build --no-cache --pull -t qbittorrent:latest .
```

The same `Dockerfile` builds both architectures; for the other one, use buildx (with QEMU on a different host):

```bash
docker buildx build --platform linux/arm64 --load -t qbittorrent:arm64 .
```

To pin versions, pass `--build-arg QBITTORRENT_VERSION=5.3.0_v2.1.2` (a [qbittorrent-nox-static](https://github.com/userdocs/qbittorrent-nox-static/releases) release without its `release-` prefix) or `--build-arg QBT_CLI_VERSION=v1.8.24285.1`.

## Credits

qBittorrent is developed by the [qBittorrent project](https://github.com/qbittorrent/qBittorrent). The static builds are by [userdocs](https://github.com/userdocs/qbittorrent-nox-static), qbittorrent-cli by [fedarovich](https://github.com/fedarovich/qbittorrent-cli). The container base, s6 init and original setup scripts are from [LinuxServer.io](https://linuxserver.io), upstream at [linuxserver/docker-qbittorrent](https://github.com/linuxserver/docker-qbittorrent).

# searxng-docker

Create a new SearXNG-SE instance in five minutes using Docker

## What is included?

| Name | Description | Docker image | Dockerfile |
| -- | -- | -- | -- |
| [SearXNG-SE](https://github.com/AeshEmi1/searxng-se) | SearXNG-SE by itself                                              | [docker.io/aeshemi1/searxng-se:latest](https://hub.docker.com/r/aeshemi1/searxng-se) | [Dockerfile](https://github.com/aeshemi1/searxng-se/blob/master/Dockerfile)               |
| [Valkey](https://github.com/valkey-io/valkey) | In-memory database                                             | [docker.io/valkey/valkey:8-alpine](https://hub.docker.com/r/valkey/valkey)        | [Dockerfile](https://github.com/valkey-io/valkey-container/blob/mainline/Dockerfile.template)             |

## How to use it
SearXNG-SE is recommended for more advanced users that already have their own reverse proxy (e.g. Nginx, HAProxy, ...).

1. [Install docker](https://docs.docker.com/install/)
2. Get searxng-se-docker
  ```sh
  cd /usr/local
  git clone https://github.com/AeshEmi1/searxng-se-docker.git
  cd searxng-se-docker
  ```
3. Generate the secret key `sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml`
4. Add your private key and public certificate under searxng/certs with the file name searxng.key and searxng.crt respectively.
5. Edit [searxng/settings.yml](https://github.com/AeshEmi1/searxng-se-docker/blob/master/searxng/settings.yml) according to your needs

> [!NOTE]
> On the first run, you must remove `cap_drop: - ALL` from the `docker-compose.yaml` file for the `searxng` service to successfully create `/etc/searxng/uwsgi.ini`. This is necessary because the `cap_drop: - ALL` directive removes all capabilities, including those required for the creation of the `uwsgi.ini` file. After the first run, you should re-add `cap_drop: - ALL` to the `docker-compose.yaml` file for security reasons.

> [!NOTE]
> Windows users can use the following powershell script to generate the secret key:
> ```powershell
> $randomBytes = New-Object byte[] 32
> (New-Object Security.Cryptography.RNGCryptoServiceProvider).GetBytes($randomBytes)
> $secretKey = -join ($randomBytes | ForEach-Object { "{0:x2}" -f $_ })
> (Get-Content searxng/settings.yml) -replace 'ultrasecretkey', $secretKey | Set-Content searxng/settings.yml
> ```

### Method 1: With Caddy included (recommended for beginners)
6. Run SearXNG in the background: `docker compose up -d`

### Method 2: Bring your own reverse proxy (experienced users)
6. Remove the caddy related parts in `docker-compose.yaml` such as the caddy service and its volumes.
7. Point your reverse proxy to the port set for the `searxng` service in `docker-compose.yml` (8080 by default).
8. Generate and configure the required TLS certificates with the reverse proxy of your choice.
9. Run SearXNG in the background: `docker compose up -d`

> [!NOTE]
> You can change the port `searxng` listens on inside the docker container (e.g. if you want to operate in `host` network mode) with the `BIND_ADDRESS` environment variable (defaults to `0.0.0.0:8080`). The environment variable can be set directly inside `docker-compose.yaml`.

## Troubleshooting - How to access the logs

To access the logs from all the containers use: `docker compose logs -f`.

To access the logs of one specific container:

- Caddy: `docker compose logs -f caddy`
- SearXNG: `docker compose logs -f searxng`
- Valkey: `docker compose logs -f redis`

### Start SearXNG with systemd

You can skip this step if you don't use systemd.
1. Copy the service template file:
   ```sh
   cp searxng-docker.service.template searxng-docker.service
   ```
  
2. Edit the content of ```WorkingDirectory``` in the ```searxng-docker.service``` file (only if the installation path is different from ```/usr/local/searxng-docker```)
   
3. Enable the service:
   ```sh
   systemctl enable $(pwd)/searxng-docker.service
   ```

4. Start the service:
   ```sh
   systemctl start searxng-docker.service
   ```

**Note:** Ensure the service file path matches your installation directory before enabling it.

## Note on the image proxy feature

The SearXNG image proxy is activated by default.

The default [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) allows the browser to access to ```${SEARXNG_HOSTNAME}``` and ```https://*.tile.openstreetmap.org;```.

If some users want to disable the image proxy, you have to modify [./Caddyfile](https://github.com/searxng/searxng-docker/blob/master/Caddyfile). Replace the ```img-src 'self' data: https://*.tile.openstreetmap.org;``` by ```img-src * data:;```.

## Multi Architecture Docker images

Supported architecture:

- amd64
- arm64
- arm/v7

## How to update ?

To update the SearXNG stack:

```sh
git pull
docker compose pull
docker compose up -d
```

Or the old way (with the old docker-compose version):

```sh
git pull
docker-compose pull
docker-compose up -d
```

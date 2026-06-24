# SkyWipe
A minimal tool for deleting lingering Bluesky accounts after a PDS migration.

## Where to find it

SkyWipe is hosted in two places:
- My website: https://skywipe.littlebitstudios.com
- nsite: https://0820eodp4mhlvosoxg9l20chwu514g3jdtuo4bjfy6i3l4d64rskywipe.nsite.lol/

## Self-host

SkyWipe is available as a Docker container from GHCR.

docker run:
```sh
docker run -p 3000:3000 ghcr.io/littlebitstudios/skywipe
```

Compose:
```yaml
services:
  main:
    restart: unless-stopped
    image: ghcr.io/littlebitstudios/skywipe
    ports:
        - 3000:3000
```
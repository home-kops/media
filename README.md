# Media

This project defines a Jenkins pipeline to deploy the media related resources on a kubernetes cluster.

## Structure

The project conists of several components:
- [Calibre](https://hub.docker.com/r/msd117/librarium)
- [Jellyfin](https://hub.docker.com/r/linuxserver/jellyfin)
- [Jellyseerr](https://hub.docker.com/r/fallenbagel/jellyseerr)
- More to come

## Ingress

All these applications are behind a Traefik reverse proxy using the Ingress route resource.

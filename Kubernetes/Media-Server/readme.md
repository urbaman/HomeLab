# Media Server

trasco: nvme1t

We will use a single pvc for configs and downloads/library, to make file transfers faster and management easier. Set it with a File CSI (NFS, CephFS) to be able to navigate through it.
After the first installation, if you set the Categories for the different apps (tv, movies, music, books, ...), create the subdirectories in the /downloads/transmission path in the PVC.

We will also create a dedicated pvc for Plex transcoding on a faster storage (nvme is better), it can be RDB type storage (Ceph).

This deployment will install:

- [x] Transmission for torrent download
- [x] Flaresolverr
- [x] Prowlarr
- [x] Sonarr
- [x] Radarr
- [x] Lidarr
- [x] Readarr
- [x] Bazaar
- [x] Unpackerr
- [x] Plex Media server with GPU transcoding
- [x] Overseer

Go to [https://www.plex.tv/claim](https://www.plex.tv/claim) and login to get the ClaimToken, enter it as the PLEX_CLAIM value and deploy the manifest.

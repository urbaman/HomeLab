# Media Server

We will use a single pvc for configs and downloads/library, to make file transfers faster and management easier. Set it with a File CSI (NFS, CephFS) to be able to navigate through it.
After the first installation, change Transmission download directories to `/downloads/transmission/complete` and `/downloads/transmission/incomplete`; if you want to set the Categories for the different apps (tv, movies, music, books, ...), create the subdirectories in the `/downloads/transmission/complete` path in the PVC, also create the `/downloads/transmission/incomplete` path.
Give them the correct (root:root, 0777) permissions through chown and chmod.

We will also create a dedicated pvc for Plex transcoding on a faster storage (nvme is better), it can be RDB type storage (Ceph).

This deployment will install:

- [x] Transmission for torrent download
- [x] Flaresolverr
- [x] Prowlarr
- [x] Sonarr
- [x] Radarr
- [x] Lidarr
- [x] Readarr
- [x] Bazarr
- [x] Unpackerr
- [x] Plex Media server with GPU transcoding
- [x] Overseer
- [x] Calibre
- [x] CalibreWS

Go to [https://www.plex.tv/claim](https://www.plex.tv/claim) and login to get the ClaimToken, enter it as the PLEX_CLAIM value and deploy the manifest.

Go to `calibre.domain.com`, go through the settings, then go to preferences, connection, check `Require username and password to access the Content Server`, add a user with read access, save, then enable the content-server (also on calibre starts).
Go to `readarr.domain.com`, and add the root folder connected to calibre content server.

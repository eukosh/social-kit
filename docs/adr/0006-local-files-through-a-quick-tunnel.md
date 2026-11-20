# Local files reach Instagram and Threads through a per-command Quick Tunnel

Instagram and Threads publish only from public URLs that Meta fetches, so a local file is served for the length of one command through an ephemeral Cloudflare Quick Tunnel (`cloudflared` with an empty `--config`, random paths), kept up through publish and closed after. Measured: Meta fetches each file once, with one plain GET during the create call. Facebook files and DM attachments go as bytes, and URLs pass through untouched. The file passes through Cloudflare on its way to Meta, and the docs say so. Without `cloudflared` the command exits 1 with `tunnel_unavailable` and the install line; skit never downloads it.

## Considered options

- **A user bucket with presigned URLs**: credentials and setup for every user; left in the fog in case Quick Tunnels prove unreliable.
- **Instagram's resumable `rupload` bytes upload**: video only, Facebook Login only, failed at 50 MB on a home connection, and Cloudflare's injected robots.txt blocks its `file_url` fetcher.
- **The unpublished Page photo CDN trick**: undocumented, and Meta rejects fbcdn URLs for Page reels.

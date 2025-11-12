# smb1-to-smb3-proxy #

This container is used to proxy an existing secure smb share (version 2+) to allow legacy devices, that only support cifs/smb v1 the access to a specific share or folder on the secure share - without downgrading the complete server to smb v1. 
Its designed to sync all files to the secure share. If a file in the legacy system changes, the modern smb server will get a request to edit the file and vice-versa. Keep in mind that smb1 is unsafe and some kind of Snapshot-System (i.e. ZFS Snapshopts) as well as a VLAN/DMZ should be used for security. The data on this share should be assumed to be attackable once an attacker is inside of your network. 

* GitHub: [skyracer2012/smb1-to-smb3-proxy](https://github.com/skyracer2012/smb1-to-smb3-proxy)
* ~~Docker Hub~~ Github Packages: [skyracer2012/smb1-to-smb3-proxy](https://github.com/skyracer2012/smb1-to-smb3-proxy/pkgs/container/smb1-to-smb3-proxy)

## Usage ##

```shell
docker pull ghcr.io/skyracer2012/smb1-to-smb3-proxy:latest
```

Example docker-compose configuration:

```yml
version: '3.7'

services:
  smb1proxy:
    image: ghcr.io/skyracer2012/smb1-to-smb3-proxy:latest
    environment:
      TZ: 'Europe/Berlin'
      USERID: 1000
      GROUPID: 1000
      SAMBA_USERNAME: scanuser
      SAMBA_PASSWORD: secret1
      GLOBAL: ntlm auth = yes
      PROXY1_ENABLE: 1
      PROXY1_SHARE_NAME: scanshare10
      PROXY1_REMOTE_PATH: //secure-host/share/path/to/folder
      PROXY1_REMOTE_DOMAIN: DOM
      PROXY1_REMOTE_USERNAME: UserA
      PROXY1_REMOTE_PASSWORD: password
      PROXY2_ENABLE: 1
      PROXY2_SHARE_NAME: scanshare30
      PROXY2_REMOTE_PATH: //other-host/share
      PROXY2_REMOTE_DOMAIN: DOM
      PROXY2_REMOTE_USERNAME: UserB
      PROXY2_REMOTE_PASSWORD: password
    ports:
      - "445:445/tcp"
    tmpfs:
      - /tmp
    restart: unless-stopped
    stdin_open: true
    tty: true
    privileged: true
```

## Config ##

The configuration is done via environment variables:

- `TZ`: Timezone
- `USERID`: Linux User ID
- `GROUPID`: Linux Group ID
- `SAMBA_USERNAME`: Global Username for the created shares
- `SAMBA_PASSWORD`: Global Password for the created shares
- `GLOBAL`: ntlm auth = yes ; Windows XP legacy support
- `PROXYx_ENABLE`: 0 = disabled, 1 = enabled
- `PROXYx_SHARE_NAME`: Samba Share name
- `PROXYx_REMOTE_PATH`: Can be just a share (//host/share) or a complete path (//host/share/path/to/folder)
- `PROXYx_REMOTE_DOMAIN`: Domain for remote path (optional)
- `PROXYx_REMOTE_USERNAME`: Username for remote path
- `PROXYx_REMOTE_PASSWORD`: Password for remote path

You can substitute x with an incremented number starting at 1 to create multiple entries. See example configuration.

# Overview

Managed to get an old Zbook working after it refused to boot. It had a damaged screen, so I yanked it out and rewired the wifi card.

![alt text](IMG-20250524-WA0002.jpeg)

Turns out it just needed some new RAM sticks and now purrs away happily. Specs:

![alt text](image.png)

## Debugging

For some reason, my local network seems to use some seriously cooked DNS configs. Had to manually change to Google DNS:

```bash
sudo vim /etc/systemd/resolved.conf

# Change:

[Resolve]
DNS=8.8.8.8 8.8.4.4
#FallbackDNS=
#Domains=
#LLMNR=yes
#MulticastDNS=yes
#DNSSEC=no
#Cache=yes
#DNSStubListener=yes
```

```bash
sudo systemctl restart systemd-resolved

# Then check fix:
ping google.com
```

## Home server setup

All services shall be added to the base README for my own reference and sanity.

### File server

I like using docker and the [File Browser](https://filebrowser.org/) project seems nice.

```bash
cd /some/folder/you/want/to/use

touch filebrowser.db
mkdir srv # this is the base dir of the web gui
vim filebrowser.json

# Add below json
```

```json
{     "port": 80,     "baseURL": "",     "address": "",     "log": "stdout",     "database": "/database.db",     "root": "/srv" }
```

```bash
vim docker-compose.yml
```

```yml
services:
  filebrowser:
    image: filebrowser/filebrowser
    container_name: filebrowser
    user: 1000:1000
    ports:
      - "8080:80"
    volumes:
      - ./srv:/srv # files will be stored here
      - ./filebrowser.db:/database.db # users info/settings will be stored here
      - ./filebrowser.json:/.filebrowser.json # config file
    restart: always
```

And then you can access this at port 8080 on your host machine. Your hostname should work, or base IP, so either of:

```bash
whoami

ip -c -h address
```

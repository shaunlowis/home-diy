# Overview

Overleaf is a nice way of rendering and editing $\LaTeX$.

Running this locally has the benefit of being in control of templates, backups and privacy.

## Setup

I prefer hosting stuff with docker compose:

```yaml
services:

  sharelatex:
    image: sharelatex/sharelatex:latest
    container_name: sharelatex
    restart: unless-stopped
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_healthy
    ports:
      - "8181:80"
    volumes:
      - sharelatex_data:/var/lib/overleaf
    environment:
      OVERLEAF_APP_NAME: "Overleaf"
      OVERLEAF_MONGO_URL: mongodb://mongo/sharelatex?directConnection=true
      OVERLEAF_REDIS_HOST: redis
      REDIS_HOST: redis
      ENABLED_LINKED_FILE_TYPES: "project_file,project_output_file"
      ENABLE_CONVERSIONS: "true"
      EMAIL_CONFIRMATION_DISABLED: "true"
      # Uncomment and configure if you want email support:
      # OVERLEAF_EMAIL_FROM_ADDRESS: "noreply@example.com"
      # OVERLEAF_EMAIL_SMTP_HOST: "smtp.example.com"
      # OVERLEAF_EMAIL_SMTP_PORT: 587
      # OVERLEAF_EMAIL_SMTP_SECURE: "false"

  mongo:
    image: mongo:8.0
    container_name: sharelatex-mongo
    restart: unless-stopped
    command: "--replSet overleaf"
    volumes:
      - mongo_data:/data/db
    healthcheck:
      test: >
        mongosh --quiet --eval
        "try { rs.status().ok } catch(e) { rs.initiate({ _id: 'overleaf', members: [{ _id: 0, host: 'mongo:27017' }] }).ok }"
      interval: 10s
      timeout: 10s
      retries: 10
      start_period: 30s

  redis:
    image: redis:6.2
    container_name: sharelatex-redis
    restart: unless-stopped
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  sharelatex_data:
  mongo_data:
  redis_data:
```

**Optional extra**: add all fonts, extras to latex install:

```bash
docker exec sharelatex wget https://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh
docker exec sharelatex sh update-tlmgr-latest.sh -- --update

# this takes forever - for me it took 45min
docker exec sharelatex tlmgr install scheme-full
```

Then you should be good to go with access on port `8181` on whatever you are hosting this on.

Lastly, don't forget to enable dark mode!

![alt text](dark_mode_local_leaf.png)

## Exporting from existing Overleaf projects

This is super simple, in your project list, click the download button in the online version.
Then click the new project button on your local version and select from zip file.

I haven't run into any issues with this, seems to work really well:

![alt text](image.png)

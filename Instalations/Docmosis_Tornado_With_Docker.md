# Docmosis Tornado 2.10.3 Installation Guide on Ubuntu 24.04

## Overview

Complete deployment of Docmosis Tornado with LibreOffice and Microsoft fonts support in a Docker container on Ubuntu 24.04.

---

## Requirements

- Docker and Docker Compose
- File `docmosisTornado2.10.3.zip` (download from the official Docmosis website)
- Ubuntu 24.04 server

---

## Step-by-Step Installation

### Step 1: Prepare Project Files

```text
project/
├── Dockerfile                     # Main Dockerfile
├── docker-compose.yml             # Docker Compose configuration
├── docmosisTornado2.10.3.zip      # Docmosis archive
└── templates/                     # Folder for .docx templates
```

---

### Step 2: Create `docker-compose.yml`

All volumes (key, site) you will get with your license. Password - set your own.

```yaml
version: '3.3'

services:
  tornado:
    build:
      context: /opt/docmosis
      dockerfile: Dockerfile
    ports:
     - "8100:8100"
    volumes:
      - /home/docmosis/templates:/opt/docmosis/created_files
    environment:
      DOCMOSIS_KEY: "your_license_key"
      DOCMOSIS_SITE: "your_docmosis_site"
      DOCMOSIS_ADMINPW: "your_passwd"
```

---

### Step 3: Create Dockerfile

Create the `Dockerfile` with the full content below (based on the official Docmosis GitHub repo —
https://github.com/Docmosis/tornado-docker/blob/master/Dockerfile, but adapted for Ubuntu configuration).

```bash
# Docmosis Tornado - Ubuntu-based Dockerfile
# LibreOffice is taken from the local directory of the host machine

FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

# Install Java, LibreOffice dependencies, fonts, and utilities
RUN apt-get update && apt-get install -y --no-install-recommends \
    openjdk-21-jre-headless \
    libcairo2 \
    libcups2 \
    libdbus-glib-1-2 \
    libglib2.0-0 \
    libsm6 \
    libxinerama1 \
    libgl1 \
    fonts-open-sans \
    fonts-freefont-ttf \
    fontconfig \
    cabextract \
    curl \
    wget \
    unzip \
    && rm -rf /var/lib/apt/lists/*

COPY libreoffice-local/ /opt/libreoffice/

# Install Microsoft core fonts
RUN echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections \
    && apt-get update && apt-get install -y --no-install-recommends \
    ttf-mscorefonts-installer \
    && rm -rf /var/lib/apt/lists/*


COPY fonts/ /usr/share/fonts/truetype/msttcore/
RUN fc-cache -f

# Create docmosis user
RUN groupadd docmosis \
    && useradd -g docmosis \
    --create-home \
    --shell /sbin/nologin \
    --comment "Docmosis user" \
    docmosis

WORKDIR /home/docmosis

ENV DOCMOSIS_VERSION=2.10.3

# Explicit filename in COPY (ENV does not work in COPY source)
COPY docmosisTornado2.10.3.zip /home/docmosis/

RUN unzip docmosisTornado${DOCMOSIS_VERSION}.zip docmosisTornado*.war docs/* licenses/* \
    && mv docmosisTornado*.war docmosisTornado.war \
    && rm -f docmosisTornado${DOCMOSIS_VERSION}.zip

RUN printf '%s\n' \
    "handlers=java.util.logging.ConsoleHandler" \
    "#Normal logging at INFO level" \
    ".level=INFO" \
    "" \
    "#Detailed logging at DEBUG level" \
    "#.level=FINE" \
    "" \
    "java.util.logging.ConsoleHandler.level=FINE" \
    "java.util.logging.ConsoleHandler.formatter=com.docmosis.webserver.launch.logging.TornadoLogFormatter" \
    'com.docmosis.webserver.launch.logging.TornadoLogFormatter.format=%1$tF %1$tT,%1$tL [%2$s] %3$s %4$s - %5$s %6$s%n' \
    > /home/docmosis/javaLogging.properties


# Add tini for proper PID 1 / zombie process handling
ENV TINI_VERSION=v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

RUN mkdir -p /home/docmosis/templates \
             /home/docmosis/workingarea \
             /home/docmosis/files_to_create \
    && chown -R docmosis:docmosis /home/docmosis/

USER docmosis

ENV DOCMOSIS_OFFICEDIR=/opt/libreoffice \
    DOCMOSIS_TEMPLATESDIR=templates \
    DOCMOSIS_WORKINGDIR=workingarea \
    LANG=C.UTF-8

ENV DOCMOSIS_ADMINPW=your_passwd

# Allow blank password (local network and host security has been configured to remove the need).
#ENV DOCMOSIS_ADMINPWALLOWBLANK=true

# Allow UNC paths in Tornado configuration. Disabled by default because of inherent security risk.
ENV DOCMOSIS_ALLOWUNCPATHS=true

EXPOSE <port>
VOLUME /home/docmosis/templates
CMD ["java", "-Dport=<port>", "-Djava.util.logging.config.file=javaLogging.properties", "-Ddocmosis.tornado.render.useUrl=http://localhost:<port>/", "-jar", "docmosisTornado.war"]
```

---

### Step 4: Build and Start

```bash
# 1. Remove old containers
docker-compose down -v

# 2. Full rebuild without cache
docker-compose build --no-cache

# 3. Start in detached mode
docker-compose up -d
```

---

### Step 5: Check Status

```bash
# View logs
docker-compose logs -f tornado

# Check container status
docker-compose ps
```

---

## Verify the Service is Running

1. Open your browser: `http://localhost:<port>`
2. Login: `admin` or no login
3. Password: `your_passwd`

---

## What Is Installed

| Component   | Version           | Path                    |
|-------------|-------------------|-------------------------|
| Ubuntu      | 24.04             | Base image              |
| LibreOffice | Ubuntu 24.04      | `/usr/lib/libreoffice`  |
| Java        | OpenJDK 21        | Headless                |
| Docmosis    | Tornado 2.10.3    | `/home/docmosis`        |
| Fonts       | MS Core + Cambria | `/usr/share/fonts`      |

---

## Environment Configuration

| Variable                  | Value                 | Description                  |
|---------------------------|-----------------------|------------------------------|
| `DOCMOSIS_OFFICEDIR`      | `/usr/lib/libreoffice`| Path to LibreOffice          |
| `DOCMOSIS_TEMPLATESDIR`   | `templates`           | Templates folder             |
| `DOCMOSIS_WORKINGDIR`     | `workingarea`         | Working directory            |
| `DOCMOSIS_ADMINPW`        | `your_passwd`         | Admin password               |
| `DOCMOSIS_ALLOWUNCPATHS`  | `true`                | Allow UNC paths              |

---

## Directory Structure

```text
project/
├── templates/           # .docx template files
├── workingarea/         # Temporary files (auto-created)
└── temp/tomcat/         # Tomcat temporary files (auto-created)
```

---

## Service Access

| Endpoint | URL                             | Description   |
|----------|---------------------------------|---------------|
| Web UI   | `http://localhost:8100`         | Admin panel   |
| API      | `http://localhost:8100/service` | REST API      |
| Port     | `8100`                          | Main port     |

---

## Encountered Issues

### Critical Build Errors

| # | Problem          | Symptom                                                  | Cause                                              | Solution                                 |
|---|------------------|----------------------------------------------------------|----------------------------------------------------|------------------------------------------|
| 1 | RPM + alien fail | `exit code: 5`                                           | Complex LibreOffice RPM dependencies on Ubuntu     | Switched to native `libreoffice` via apt |
| 2 | Docker warning   | `NoEmptyContinuation: Empty continuation line (line 24)` | Incorrect `\` line continuation in Dockerfile      | Fixed RUN command formatting             |
| 3 | URL markdown     | `[http://...](http://...)` instead of `http://...`       | Markdown brackets included in wget URL             | Removed brackets from URL                |

---

### Docmosis Startup Errors

| # | Problem                   | Symptom                                        | Cause                                                   | Solution                                                |
|---|---------------------------|------------------------------------------------|---------------------------------------------------------|---------------------------------------------------------|
| 4 | `temp/tomcat` permissions | `Unable to create the directory [temp/tomcat]` | User `docmosis` had no write access to `/home/docmosis` | `mkdir -p temp/tomcat` + `chown` BEFORE `USER docmosis` |
| 5 | Duplicate packages        | Extra `apt-get update` and `cabextract` calls  | Re-installing already installed packages                | Removed duplicates                                      |

---

### Compatibility Issues

| # | Problem               | Symptom                    | Cause                                     | Solution                                                |
|---|-----------------------|----------------------------|-------------------------------------------|---------------------------------------------------------|
| 6 | LibreOffice path      | Docmosis could not find LO | Incorrect path in `DOCMOSIS_OFFICEDIR`    | Used `/usr/lib/libreoffice` (Ubuntu) instead of `/opt/` |
| 7 | soffice symlink       | Docmosis could not see LO  | `/usr/local/bin/soffice` was missing      | `ln -sf /usr/bin/libreoffice /usr/local/bin/soffice`    |
| 8 | Java-LibreOffice      | LO launched without Java   | `libreoffice-java-common` was missing     | Added the package                                       |

---

### Configuration Issues

| #  | Problem               | Symptom                                         | Cause                                     | Solution                                           |
|----|-----------------------|-------------------------------------------------|-------------------------------------------|----------------------------------------------------|
| 9  | Docker Compose logs   | `KeyError: 'id'` in Python                      | docker-compose bug (does not affect work) | Ignore (cosmetic issue)                            |
| 10 | ENV variables in COPY | Docker did not substitute `${DOCMOSIS_VERSION}` | ENV does not work in COPY source          | Used explicit filename `docmosisTornado2.10.3.zip` |
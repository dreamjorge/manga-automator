# Automated Manga Download and Management Project for "Hunter x Hunter" from MangaDex

This project uses **Docker Compose** to integrate **qBittorrent**, **FlexGet**, and **Calibre** to automate the download and management of the manga  from MangaDex. Additionally, it configures an OPDS feed in Calibre so you can access your library from **KooReader**.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Structure](#project-structure)
3. [`docker-compose.yml` File](#docker-composeyml-file)
4. [Configuring FlexGet for manga](#configuring-flexget-for-manga)
5. [Calibre Configuration](#calibre-configuration)
6. [Starting the Services](#starting-the-services)
7. [Accessing Web Interfaces](#accessing-web-interfaces)
8. [Automating and Monitoring MangaDex](#automating-and-monitoring-mangadex)
9. [Security and Ethical Considerations](#security-and-ethical-considerations)
10. [Conclusion and Next Steps](#conclusion-and-next-steps)

---

## Prerequisites

Before getting started, ensure you have:

- **Docker** and **Docker Compose** installed:
  - [Docker Installation Guide](https://docs.docker.com/get-docker/)
  - [Docker Compose Installation Guide](https://docs.docker.com/compose/install/)

- Basic knowledge of Docker and YAML files.

- Access to MangaDex (having an account is recommended but not mandatory).

---

## Project Structure

Organize your project as follows:

```
manga-automator/
├── calibre/
│   ├── config/         # Calibre persistent configuration files
│   └── library/        # Calibre library (manga files will be stored here)
├── flexget/
│   └── config/         # FlexGet configuration file
├── qbittorrent/
│   └── config/         # qBittorrent persistent configuration files
├── downloads/          # Shared folder for downloads
└── docker-compose.yml  # Docker Compose orchestration file
```

Create the necessary folders:

```bash
mkdir -p manga-automator/{calibre/{config,library},flexget/config,qbittorrent/config,downloads}
```

---

## `docker-compose.yml` File

Create the `docker-compose.yml` file in the project's root (`manga-automator`) with the following content:

```yaml
version: '3.8'

services:
  qbittorrent:
    image: linuxserver/qbittorrent
    container_name: qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
      - WEBUI_PORT=8080
      - QBITTORRENT_WEBUI_PASSWORD=your_secure_password
    volumes:
      - ./qbittorrent/config:/config
      - ./downloads:/downloads
    ports:
      - 8080:8080
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped

  flexget:
    image: flexget/flexget
    container_name: flexget
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./flexget/config:/config
      - ./downloads:/downloads
    depends_on:
      - qbittorrent
    restart: unless-stopped
    command: web

  calibre:
    image: linuxserver/calibre
    container_name: calibre
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
      - CALIBRE_LIBRARY=/calibre/library
    volumes:
      - ./calibre/config:/config
      - ./calibre/library:/calibre/library
      - ./downloads:/downloads
    ports:
      - 8081:8080
    restart: unless-stopped
```

> **Note:** Change `your_secure_password` to a strong password.

---

## Starting the Services

From the project's root (`manga-automator`), run:

```bash
docker-compose up -d
```

Verify that the containers are running:

```bash
docker-compose ps
```

To manually test FlexGet, run:

```bash
docker-compose exec flexget flexget execute
```

---

## Accessing Web Interfaces

- **qBittorrent:**  
  URL: `http://<SERVER_IP>:8080`  
  Credentials:  
  - Username: `admin`  
  - Password: `your_secure_password`

- **FlexGet (Web Interface):**  
  URL: `http://<SERVER_IP>:5050`  

- **Calibre:**  
  URL: `http://<SERVER_IP>:8081`  
  OPDS Feed: `http://<SERVER_IP>:8081/opds/`

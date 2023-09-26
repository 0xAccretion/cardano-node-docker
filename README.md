# Cardano Node Docker Repository

[![Build status badge](https://github.com/0xAccretion/cardano-node-docker/actions/workflows/main.yml/badge.svg)](https://github.com/0xAccretion/cardano-node-docker/actions/workflows/main.yml)
[![Docker Pulls](https://img.shields.io/docker/pulls/0xaccretion/cardano-node-arm64-1-35-7)](https://hub.docker.com/r/0xaccretion/cardano-node-arm64-1-35-7)
[![License](https://img.shields.io/github/license/0xAccretion/cardano-node-docker)](https://github.com/0xAccretion/cardano-node-docker/blob/main/LICENSE.md)



This repository contains Dockerfiles and scripts for running a Cardano node on ARM64 architectures. It aims to make the setup and operation of Cardano nodes as seamless as possible.

## Features

- 📦 **Dockerized**: Everything is containerized for easy setup and management.
- 🌐 **Network Configurable**: Supports different Cardano networks (Mainnet, Testnet).
- 💡 **Extensible**: Easily extendable with additional services like transaction submission APIs.
- 🌟 **Latest Versions**: Always up-to-date with the latest versions of Cardano Node.

## Quick Start

1. **Clone the repository:**

    ```bash
    git clone https://github.com/0xAccretion/cardano-node-docker.git
    ```

2. **Navigate to the repository folder:**

    ```bash
    cd cardano-node-docker
    ```

3. **Build the Docker image:**

    ```bash
    docker build -t cardano-node:latest .
    ```

4. **Run using Docker Compose:**

    ```bash
    docker-compose up -d
    ```

## Configuration

- Modify the `docker-compose.yaml.example` file to suit your needs. By default, it is set up for running a Cardano mainnet node.
- Environment variables can be set in the `.env` file.

```yml
version: "3.3"

services:
  cardano-node:
    network_mode: host
    container_name: cardano-node
    image: 0xaccretion/cardano-node-arm64-1-35-7:latest # Replace with your own image
    volumes:
      - cardano-node-db:/data/db
      - cardano-node-ipc:/ipc
      - ./config:/config # Make sure you place your configuration files here
    restart: on-failure
    environment:
      - CARDANO_NODE_SOCKET_PATH=/ipc/node.socket
    ports:
      - "3000:3000"
      - "12788:12788"
      - "12798:12798"
    command:
      - "run"
      - "--config /config/mainnet-config.json"
      - "--topology /config/mainnet-topology.json"
      - "--database-path /data/db"
      - "--socket-path /ipc/node.socket"
      - "--port 3000"
    healthcheck:
      test: ["CMD-SHELL", "curl -f 127.0.0.1:12788 || exit 1"]
      interval: 60s
      timeout: 10s
      retries: 5
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"

 cardano-submit-api:
    image: 0xaccretion/cardano-node-arm64-1-35-7:latest # Replace with your own image
    volumes:
      - ./config:/config
      - cardano-node-ipc:/ipc
    restart: on-failure
    depends_on:
      - cardano-node
    command:
      - "submit-api"
      - "--config /config/submit-api-config.json" # Replace with your specific config file if needed
      - "--socket-path /ipc/node.socket"
      - "--listen-address 0.0.0.0"
      - "--port 8090"
    ports:
      - "8090:8090"
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"

volumes:
  cardano-node-db:
    driver: local
  cardano-node-ipc:
    driver: local
```

### Command-line options

You can control the behavior of the Cardano node and associated services using command-line options:

```bash
docker run cardano-node:latest run ...
docker run cardano-node:latest cli ...
docker run cardano-node:latest submit-api ...
```
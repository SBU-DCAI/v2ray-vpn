V2Ray + Ubuntu Test Environment Documentation
Overview

This Docker setup provides an easy-to-use test environment for V2Ray. It allows you to run a V2Ray proxy server and a separate Ubuntu container for testing network connectivity through the proxy. The setup supports multiple protocols, including:

VLESS

VMess

Trojan

Shadowsocks

Components

V2Ray Container
Runs the V2Ray server with a user-provided configuration. Exposes SOCKS5 and optional HTTP proxies for testing.

Ubuntu Test Container
A minimal Ubuntu environment configured to route all traffic through the V2Ray proxy. Can be used to test connectivity, download files, or verify blocked websites.

Docker-Compose File
version: "3.9"

services:
  v2ray:
    image: teddysun/v2ray
    container_name: v2ray
    restart: unless-stopped
    networks:
      - v2ray-net
    volumes:
      - ./config.json:/etc/v2ray/config.json:ro
    ports:
      - "1080:1080" # SOCKS5 proxy
      - "1081:1081" # Optional HTTP proxy
    environment:
      - TZ=Asia/Tehran

  ubuntu-test:
    image: ubuntu:24.04
    container_name: ubuntu-test
    stdin_open: true
    tty: true
    networks:
      - v2ray-net
    environment:
      - ALL_PROXY=socks5://v2ray:1080
    command: /bin/bash

networks:
  v2ray-net:
    driver: bridge

Explanation of the Compose File
1. Version
version: "3.9"


Specifies the Docker Compose version. Version 3.9 is compatible with modern Docker versions and supports networking, volumes, and restart policies.

2. Services
a) V2Ray

image: Uses teddysun/v2ray, a prebuilt V2Ray image.

container_name: Readable container name.

restart: unless-stopped: Automatically restarts if the container crashes.

networks: Connects the container to a custom bridge network.

volumes: Mounts config.json from the host. :ro makes it read-only inside the container.

ports: Exposes SOCKS5 (1080) and optional HTTP (1081) proxy ports.

environment: Sets environment variables like timezone.

This container runs the V2Ray server supporting VLESS, VMess, Trojan, and Shadowsocks protocols.

b) Ubuntu Test Container

image: Minimal Ubuntu image.

stdin_open & tty: Keeps the container interactive.

networks: Connected to the same network as V2Ray.

environment: Sets ALL_PROXY so CLI tools automatically use the V2Ray proxy.

command: Starts Bash for interactive testing.

This container is used to test network requests through V2Ray without affecting the host machine.

3. Networks
networks:
  v2ray-net:
    driver: bridge


Defines a custom bridge network allowing the containers to communicate by name (v2ray) without exposing all internal ports externally.

Usage

Place your config.json in the same folder as the docker-compose.yml.

Run:

docker-compose up -d


Access the Ubuntu container:

docker exec -it ubuntu-test bash


Test network connectivity using curl, wget, or other CLI tools:

curl https://www.google.com


All traffic from the Ubuntu container is routed through the V2Ray proxy automatically.

Notes

Supports VLESS, VMess, Trojan, and Shadowsocks protocols.

The Ubuntu container can be extended with additional testing tools as needed.

Ports 1080 (SOCKS5) and 1081 (HTTP) are optional and can be modified in docker-compose.yml.
(but the default port is on 1080 if you want to change it you shoud change the v2rayjson.py too)

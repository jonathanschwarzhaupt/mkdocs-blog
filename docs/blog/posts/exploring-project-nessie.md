---
date:
  created: 2025-04-23
tags:
  - Iceberg
  - Project Nessie
categories:
  - Data Engineering
links:
- External links:
  - Project Nessie Downloads: https://projectnessie.org/downloads/
  - Docker Compose Profiles: https://docs.docker.com/compose/how-tos/profiles/
  - Nessie CLI Reference: https://projectnessie.org/nessie-0-103-3/cli/
  - Intro to PyIceberg: https://www.dremio.com/blog/intro-to-pyiceberg/
  - PyIceberg API Documentation: https://py.iceberg.apache.org/api/
---

# Exploring Project Nessie - a transactional catalogue over iceberg tables

This week, I dove into Project Nessie - an open-source transactional data catalogue for Apache Iceberg tables. <!-- more --> I'd heard about Nessie's git-like semantics and was curious about its potential for better managing data versioning and auditability in my projects.

## Docker compose setup for Nessie Server and CLI

To experiment locally, I leveraged Docker, conveniently supported by a guide provided by the Nessie team. Following their materials, I put together a straightforward Docker Compose file that neatly places both the Nessie server and CLI into the same Docker network. This setup greatly simplifies communication between the containers.

Here's a quick excerpt from my `docker-compose.yaml`, the repository can be found on [GitHub](https://github.com/jonathanschwarzhaupt/lab/tree/main/docker/project-nessie):

```yaml
services:
  nessie-server:
    image: ghcr.io/projectnessie/nessie:0.103.3
    container_name: nessie-server
    ports:
      - "19120:19120"
      - "9000:9000"

  nessie-cli:
    image: ghcr.io/projectnessie/nessie-cli:0.103.3
    container_name: nessie-cli
    stdin_open: true   # -i
    tty: true          # -t
    profiles:
      - cli
    depends_on:
      - nessie-server
```

With everything configured, I spun up the server with `docker compose up -d`, and then initiated an interactive CLI session using `docker compose run nessie-cli`. I particularly liked how Docker Compose's `profiles` feature ensures only the services needed at runtime are started.

Inside the Nessie CLI, connecting to the server was straightforward:

```bash
CONNECT TO http://nessie-server:19120/api/v2
```

I explored some basic commands:
- `create namespace my_new_namespace`: This felt similar to creating folders or database schemas.
- `create branch if not exists my_new_branch from main`: Like git branches, this allows for isolated, version-controlled experiments with data.
- `use branch my_new_branch`: Activates the new branch, making any subsequent operations exclusive to it. The REPL conveniently marks the active branch.

Checking the Nessie UI at `http://127.0.0.1:19120`, it was clear that namespaces created on the branch were entirely isolated from `main`. Exiting the CLI was as easy as typing `exit`.

## PyIceberg and Marimo Notebooks

Next up was experimenting with PyIceberg to populate our data catalogue. After setting up a Python virtual environment and installing dependencies, I used [Marimo](https://docs.marimo.io/) — a fantastic interactive Python environment — to test various operations like creating Iceberg tables using PyIceberg schemas and even PyArrow schemas, which seem particularly useful for ETL scenarios.

I encountered an initial snag: when writing data to the catalogue, PyIceberg threw connection errors related to the `minio:9000` endpoint. The fix involved tweaking my Docker Compose configuration to add an `external-endpoint` setting for the `nessie-server` service, enabling proper communication between services and clients.

## Nessie for ETL pipelines

Looking ahead, I'm particularly excited about Nessie's potential for implementing a write-audit-publish pattern. I can already envision a process where Airflow tasks fetch new data, branch off for isolated changes, perform quality checks, and only then merge into the main data branch — maintaining robust control and auditability.

---
date:
  created: 2025-07-28
tags:
  - Docker
  - GitHub
categories:
  - DevOps
---

# Manually push to GitHub Container registry with Docker login

Today I wanted to manually push a Docker image I built locally to my github container registry for debugging purposes.

<!-- more -->

I built the image like so

```bash
docker build --pull --tag ghcr.io/jonathanschwarzhaupt/lab-airflow:v0.0.1 .
```

Important to not forget the dot in the end to indicate that the Dockerfile resides in the current directory.

I had issues pushing the image to ghcr.io, however. I confirmed that I was logged in as the correct user through

```bash
gh auth status

gh auth switch
```

But it still would not work. I needed to create a new personal access token under my Github user's developer settings, granting it the packages:write (and :delete) permissions.

Then I logged in to docker using this nifty command:

```bash
echo "my-pasted-token" | docker login ghcr.io --username <my-username> --password-stdin
```

Finally, I could run

```bash
docker push ghcr.io/jonathanschwarzhaupt/lab-airflow:v0.0.1
```

successfully.

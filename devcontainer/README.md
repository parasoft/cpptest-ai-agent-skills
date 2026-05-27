# C/C++test Docker Image

This folder contains a `Dockerfile` for building a Debian-based image with:
- Parasoft C/C++test (Standard and optionally CT)
- C/C++test extension for VS Code (if present in the distribution)
- GitHub Actions Runner
- GitHub CLI
- OpenAI Codex CLI

## Prerequisites

- Place the C/C++test distribution files matching `parasoft_cpptest_*.tar.gz` in the same directory as this Dockerfile before building the image.
- Prepare your Parasoft license server URL and your OpenAI API token for the build command.

## Build

```bash
docker build --build-arg OPEN_API_KEY=YOURKEY --build-arg LICENSE_SERVER_URL=https://ls.company.com:8443 -t cpptest:latest .
```

SECURITY WARNING:  
`OPEN_API_KEY` will be stored in the image in plain text, so make sure to use a key with limited permissions and do not share the image with untrusted parties, or update the Dockerfile to pass the key at runtime only.

## Run

```bash
docker run -it --rm cpptest:latest bash
```

## GitHub Runner (optional)

Inside the container:

```bash
cd actions-runner
./config.sh --url https://github.com/my/project --token YOURTOKENFROMGITHUB
./run.sh
```

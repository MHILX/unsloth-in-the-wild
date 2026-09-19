# unsloth-in-the-wild

Local Docker setup for running Unsloth with a Zscaler-inspected TLS connection.

## Prerequisites

- Docker Desktop with Docker Compose support
- NVIDIA Container Toolkit and a supported NVIDIA GPU
- A DER-encoded Zscaler root CA certificate

## Setup

1. Place the Zscaler certificate at `certs/zscaler-root-ca.cer`.

	The certificate is intentionally ignored by Git and must be supplied separately on each machine.

2. Build the image:

	```powershell
	docker compose build
	```

3. Start the container:

	```powershell
	docker compose up -d
	```

The local `work/` directory is mounted at `/workspace/work` in the container.

## Local services

- Jupyter: `http://localhost:8888`
- Unsloth Studio: `http://localhost:8000`
- SSH: `localhost:2222`

The Jupyter and Unsloth Studio passwords are defined in `docker-compose.yml` for local development only. Do not reuse them for shared or production environments.

Stop the container with:

```powershell
docker compose down
```

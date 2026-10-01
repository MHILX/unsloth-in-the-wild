# unsloth-in-the-wild

Local Docker setup for running Unsloth with a Zscaler-inspected TLS connection.

## Prerequisites

- Docker Desktop with Docker Compose support
- NVIDIA Container Toolkit and a supported NVIDIA GPU
- A DER-encoded Zscaler root CA certificate

## Setup

1. Place the Zscaler certificate at `certs/zscaler-root-ca.cer`.

	The certificate is intentionally ignored by Git and must be supplied separately on each machine.

2. Update the base image, build the image, and start the container:

	```powershell
	docker compose build --pull && docker compose up -d
	```

The local `work/` directory is mounted at `/workspace/work` in the container.

The Compose project name is fixed to `unsloth`, matching the existing service. This prevents a container-name conflict when running Compose from this repository instead of its previous directory.

## Notebooks

Notebook data is consolidated in the local `notebooks/` folder, beside [docker-compose.yml](docker-compose.yml). The complete source tree, including notebooks, helper scripts, assets, and synchronization metadata, was copied from the existing container and verified with SHA-256 checksums.

The folder is mounted read-write at `/workspace/unsloth-notebooks`. The main notebooks are in `notebooks/nb/` on Windows and `/workspace/unsloth-notebooks/nb/` in Jupyter. Once the mount is applied, edits are saved directly to the Windows files and survive container recreation.

`/workspace/Unsloth Notebooks` is only a categorized symbolic-link view, not a separate set of notebook files. It was removed during consolidation, but the image or startup scripts may restore it when the container is recreated. Use `/workspace/unsloth-notebooks/nb/` for the canonical files.

The copied `notebooks/` tree is ignored by Git. Back it up separately, and keep the host folder when recreating containers or relocating this repository.

## Model locations

### Local GGUF files

[docker-compose.yml](docker-compose.yml) mounts `%USERPROFILE%\Models` on Windows at `/workspace/models` in the container. The mount is read-only: add or copy model files into the Windows folder, not through the container.

On this laptop, the copied Gemma model is located at:

```text
C:\Users\Mohammed.Hoque\Models\Gemma\gemma-4-12B-it-qat-UD-Q4_K_XL.gguf
```

Unsloth Studio sees that same file at:

```text
/workspace/models/Gemma/gemma-4-12B-it-qat-UD-Q4_K_XL.gguf
```

Studio's custom-model directory is `/workspace/models`. Use the container path when selecting a local model in Studio. The bind mount exposes the Windows file without making another copy. Changes to mounts require recreating the container; restarting it alone does not apply them.

### Downloaded model cache

Hugging Face downloads inside the container are stored at `/workspace/.cache/huggingface/hub`, backed by the Docker volume `unsloth_huggingface`. They are not automatically saved in the Windows `Models` folder or the project's `work/` directory. Copying a model into `Models` creates a separate file and uses additional disk space; the original remains in the Docker cache until explicitly removed.

The duplicate main Gemma GGUF was removed from the Docker cache after the Windows copy was verified. Its smaller projector and MTP companion files remain cached. Select the local model path shown above to use the retained Windows file rather than downloading the main model again.

The cache uses your laptop's disk space, but its files live inside Docker Desktop's Linux virtual disk. On this machine, that disk is `%LOCALAPPDATA%\Docker\wsl\disk\docker_data.vhdx`. Browse the actual cached model files through **Docker Desktop > Volumes > unsloth_huggingface > hub**, rather than trying to open the virtual disk directly.

The Windows Hugging Face cache at `%USERPROFILE%\.cache\huggingface\hub` is separate from the Docker cache and is not mounted by this Compose configuration.

Studio settings and data are stored separately in the `unsloth_studio_home` volume, mounted at `/opt/unsloth-studio`. Both named volumes are declared external and must already exist before starting this Compose setup. Recreating or removing the container preserves them; deleting the volumes removes their stored data.

## Zscaler certificate

`Dockerfile.zscaler` extends the Unsloth image and adds the supplied Zscaler root certificate to the container's trusted certificate store. This allows HTTPS tools inside the container to trust certificates re-signed by Zscaler during TLS inspection.

It does not install a Zscaler client or VPN. The certificate is converted from DER to PEM format during the image build, and the temporary copy is removed afterward. Because the certificate is ignored by Git, it must be provided separately before building.

## Local services

- Jupyter: `http://localhost:8888`
- Unsloth Studio: `http://localhost:8000`
- SSH: `localhost:2222`

The Jupyter and Unsloth Studio passwords are defined in `docker-compose.yml` for local development only. Do not reuse them for shared or production environments.

Stop the container with:

```powershell
docker compose down
```

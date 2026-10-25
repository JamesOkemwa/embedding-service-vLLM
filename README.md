# vLLM Embedding Model

A Docker Compose setup that serves [BAAI/bge-m3](https://huggingface.co/BAAI/bge-m3) embeddings through [vLLM](https://github.com/vllm-project/vllm)'s OpenAI-compatible API.

## Requirements

- Docker with Compose v2
- An NVIDIA GPU with current drivers (tested on an RTX 4050, 6 GB)
- On Windows: Docker Desktop with the WSL2 backend

## Usage

Start the service:

```sh
docker compose up -d
```

The first start downloads the model and can take a few minutes. The model is cached in the `hf-cache` volume. Check progress with `docker compose logs -f` or `docker compose ps`; the container is ready when it reports `healthy`.

Request embeddings:

```sh
curl http://127.0.0.1:8000/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model": "BAAI/bge-m3", "input": ["Hello, world!"]}'
```

The response contains one 1024-dimensional vector per input string.

Stop the service:

```sh
docker compose down
```

## Configuration

Settings live in [compose.yaml](compose.yaml):

| Setting | Value | Notes |
| --- | --- | --- |
| Port | `127.0.0.1:8000` | Bound to localhost only. Change the mapping to expose it on your network. |
| `--max-model-len` | `8192` | BGE-M3's full context length. |
| `--gpu-memory-utilization` | `0.75` | Lower it if startup fails with a memory error or you share the GPU. |
| Health check | `GET /health` | 300s start period to allow for the model download. |

The service has no authentication. Add `--api-key <key>` to the `command` list before exposing it beyond localhost.

## Notes

- Only open Hugging Face models are supported as configured. No `HF_TOKEN` is set.
- The image is pinned to `vllm/vllm-openai:v0.32.0` for reproducible builds. The pooling flags used here (`--runner`, `--pooler-config.task`, `--hf-overrides`) have changed between vLLM releases, so re-test after changing the version.

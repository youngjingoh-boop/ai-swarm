# Backend-only image for MiroFish (jason-1000-agents)
# Build context = the backend/ directory (rootDir: backend on Render / Railway).
FROM python:3.11-slim

# System libs some Python deps need at runtime
RUN apt-get update \
  && apt-get install -y --no-install-recommends build-essential \
  && rm -rf /var/lib/apt/lists/*

# uv (Python package manager) from the official image
COPY --from=ghcr.io/astral-sh/uv:0.9.26 /uv /uvx /bin/

WORKDIR /app

# Install deps first for layer caching
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen

# App source
COPY . .

# Production defaults (never run the Werkzeug debugger on a public host)
ENV FLASK_DEBUG=False \
    FLASK_HOST=0.0.0.0 \
    PYTHONUNBUFFERED=1

EXPOSE 5001

# The host injects $PORT; run.py reads FLASK_PORT (falls back to 5001 locally).
CMD ["sh", "-c", "FLASK_PORT=${PORT:-5001} uv run python run.py"]

# ── Stage 1: builder ──────────────────────────────────────────────────────────
FROM python:3.10-slim AS builder

WORKDIR /build

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy only requirements first (better layer caching)
COPY ./requirements.txt .

# Install all packages into an isolated directory
RUN pip install --no-cache-dir --prefix=/install \
    torch==2.11.0 --extra-index-url https://download.pytorch.org/whl/cpu

RUN pip install --no-cache-dir --prefix=/install \
    --upgrade --timeout=300 -r requirements.txt


# ── Stage 2: runtime ──────────────────────────────────────────────────────────
FROM python:3.10-slim AS runtime

WORKDIR /code

# Copy only the installed packages from builder
COPY --from=builder /install /usr/local

# Copy application code
COPY . /code

# Run as non-root user (security best practice)
RUN useradd -m appuser && chown -R appuser /code
USER appuser

CMD ["fastapi", "run", "main.py", "--port", "80"]
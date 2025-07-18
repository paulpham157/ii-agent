# Use a Python image with uv pre-installed
FROM ghcr.io/astral-sh/uv:python3.12-bookworm-slim

# Install the project into `/app`
WORKDIR /app

# Enable bytecode compilation
ENV UV_COMPILE_BYTECODE=1

# Copy from the cache instead of linking since it's a mounted volume
ENV UV_LINK_MODE=copy

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    curl \
    gnupg \
    lsb-release \
    tmux \
    ffmpeg \
    xvfb \
    git \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    && rm -rf /var/lib/apt/lists/*

# Install code-server
RUN curl -fsSL https://code-server.dev/install.sh | sh
# Install the project's dependencies using the lockfile and settings
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project --no-dev --prerelease=allow

# Install Playwright in a single layer
RUN uv run playwright install --with-deps chromium

# Then, add the rest of the project source code and install it
# Installing separately from its dependencies allows optimal layer caching
COPY . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-dev --prerelease=allow

# Place executables in the environment at the front of the path
ENV PATH="/app/.venv/bin:$PATH"

ENV FILE_STORE_PATH=/.ii_agent

# Set environment variables
ENV PYTHONUNBUFFERED=1

RUN mkdir -p $FILE_STORE_PATH

# Expose port for WebSocket server
EXPOSE 8000

# Reset the entrypoint, don't invoke `uv`
ENTRYPOINT []

# Run the application
CMD ["xvfb-run", "--auto-servernum", "python", "ws_server.py"]

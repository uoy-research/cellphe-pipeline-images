FROM python:3.12.12-slim-trixie AS builder

RUN apt-get update && apt-get install -y curl
RUN rm -rf /var/cache/apt/archives /var/lib/apt/lists/*

# Install uv purely for speed
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
 && mv /root/.local/bin/uv /usr/local/bin/uv

# Install cellpose without any dependencies
RUN uv pip install --system --break-system-packages cellpose==4.0.8

# Pull in custom models
COPY models/* /cellpose/

ENTRYPOINT ["/bin/bash"]

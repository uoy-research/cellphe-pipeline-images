FROM python:3.12.12-slim-trixie AS builder

RUN apt update && apt install -y curl && rm -rf /var/cache/apt/archives /var/lib/apt/lists/*.

# Install uv purely for speed
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
 && mv /root/.local/bin/uv /usr/local/bin/uv

# Install CPU only torch
RUN uv pip install --system --break-system-packages torch --index-url https://download.pytorch.org/whl/cpu
# Now installl cellphe
RUN uv pip install --system --break-system-packages cellphe

# Copy python dependencies over from builder
FROM python:3.12.12-slim-trixie AS app
COPY --from=builder /usr/local/lib/python3.12 /usr/local/lib/python3.12
ENTRYPOINT ["/bin/bash"]

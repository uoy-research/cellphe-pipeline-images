FROM debian:trixie-20251103-slim

# Install system packages
RUN apt update && apt install -y zip libpng-dev libjpeg-dev libtiff-dev libtiff-tools imagemagick && rm -rf /var/cache/apt/archives /var/lib/apt/lists/*

ENTRYPOINT ["bash"]

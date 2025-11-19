# ---- Stage 1: Builder ----
FROM alpine:3.22.2 AS builder

# Install all required tools and dependencies
RUN apk add --no-cache \
    bash \
    imagemagick \
    zip \
    tiff-tools

# ---- Stage 2: Minimal runtime ----
# Only saves 4MB due to copying libraries over, but libraries
# are nearly only those used for these utilities
FROM alpine:3.22.2

# Copy only the executables and required shared libraries from builder
COPY --from=builder /bin/bash /bin/bash
COPY --from=builder /usr/bin/zip /usr/bin/zip
COPY --from=builder /usr/bin/tiff* /usr/bin/
COPY --from=builder /usr/bin/magick /usr/bin/magick
COPY --from=builder /usr/lib /usr/lib

ENTRYPOINT ["/bin/bash"]

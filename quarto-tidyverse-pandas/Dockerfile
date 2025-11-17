# ---- Stage 1: Quarto builder ----
FROM alpine:3.22.2 AS quarto-builder

RUN apk add --no-cache curl tar gzip

# Install Quarto CLI (official release)
ARG QUARTO_VERSION=1.8.25
RUN curl -L "https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz" \
    | tar -xz -C /opt \
 && mv /opt/quarto-${QUARTO_VERSION} /opt/quarto

# ---- Stage 2: R + packages builder ----
FROM rhub/r-minimal:4.5.2 AS r-builder

# Using installr to make it one call
# Plus all system dependencies are listed in r-minimal's docs
# https://github.com/r-hub/r-minimal/blob/main/examples/tidyverse-minimal/Dockerfile
RUN installr -d \
    -t "gfortran" \
    dplyr ggplot2 knitr rmarkdown reticulate Cairo

# Remove large documentation as detailed in r-minimal docs
# https://github.com/r-hub/r-minimal/blob/main/examples/tidyverse-minimal/Dockerfile
RUN rm -rf /usr/local/lib/R/library/*/help && \
    rm -rf /usr/local/lib/R/library/*/NEWS* && \
    rm -rf /usr/local/lib/R/library/*/html && \
    rm -rf /usr/local/lib/R/library/*/doc && \
    rm -rf /usr/local/lib/R/library/data.table/tests && \
    rm -rf /usr/local/lib/R/library/knitr/examples && \
    rm -rf /usr/local/lib/R/library/lubridate/pkgdown && \
    rm -rf /usr/local/lib/R/library/googledrive/extdata && \
    rm -rf /usr/local/lib/R/library/colorspace/*/www && \
    rm -rf /usr/local/lib/R/library/vroom/bench

# ---- Stage 3: Python + uv builder ----
FROM alpine:3.22.2 AS python-builder

RUN apk add --no-cache python3=3.12.12-r0 curl build-base python3-dev

# Install uv purely for speed
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
 && mv /root/.local/bin/uv /usr/local/bin/uv

# Install numpy and pandas via uv
RUN uv pip install --system --break-system-packages numpy pandas matplotlib scikit-image pillow

# ---- Stage 4: Get libc library for Deno ----
FROM gcr.io/distroless/cc AS cc

# ---- Stage 5: Final runtime ----
FROM rhub/r-minimal:4.5.2

# Runtime dependencies
# TODO can any of these be removed?
RUN apk add --no-cache \
      bash \
      python3=3.12.12-r0 \
      fontconfig \
      ttf-dejavu \
      tini \
      libstdc++ \
      libgfortran \
      libgcc \
      libcurl \
      libxml2 \
      fribidi \
      harfbuzz \
      freetype \
      libpng \
      tiff \
      libjpeg \
      icu-libs \
      cairo-dev

# Copy R packages
COPY --from=r-builder /usr/local/lib/R /usr/local/lib/R

# Copy Python packages
COPY --from=python-builder /usr/lib/python3.12 /usr/lib/python3.12

# Copy libc libraries needed for deno (quarto dependency)
# Approach taken from link below with modification as if take all libraries it clashes with R install
# https://github.com/orgs/quarto-dev/discussions/12872#discussioncomment-13354199
COPY --from=cc --chown=root:root --chmod=755 /lib/*-linux-gnu/ld-linux-* /usr/local/lib/
COPY --from=cc --chown=root:root --chmod=755 /lib/*-linux-gnu/libdl.so* /usr/local/lib/
COPY --from=cc --chown=root:root --chmod=755 /lib/*-linux-gnu/librt.so* /usr/local/lib/
COPY --from=cc --chown=root:root --chmod=755 /lib/*-linux-gnu/libpthread.so* /usr/local/lib/
COPY --from=cc --chown=root:root --chmod=755 /lib/*-linux-gnu/libm.so* /usr/local/lib/
COPY --from=cc --chown=root:root --chmod=755 /lib/*-linux-gnu/libc.so* /usr/local/lib/
RUN mkdir /lib64 && ln -s /usr/local/lib/ld-linux-* /lib64/
ENV LD_LIBRARY_PATH="/usr/local/lib"

# Place quarto binaries in PATH
COPY --from=quarto-builder /opt/quarto /opt/quarto
RUN ln -s /opt/quarto/bin/quarto /usr/local/bin/quarto
RUN ln -s /opt/quarto/bin/tools/x86_64/deno /usr/local/bin/deno
RUN ln -s /opt/quarto/bin/tools/x86_64/pandoc /usr/local/bin/pandoc

WORKDIR /app

ENTRYPOINT ["/bin/bash"]

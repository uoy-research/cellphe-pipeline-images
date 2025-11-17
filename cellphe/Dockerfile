FROM alpine:3.22.2 AS builder

RUN apk add --no-cache python3=3.12.12-r0 curl build-base python3-dev geos-dev

# Install uv purely for speed
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
 && mv /root/.local/bin/uv /usr/local/bin/uv

# Install cellphe without any dependencies
RUN uv pip install --system --break-system-packages --no-deps cellphe
# Install dependencies manually
RUN uv pip install --system --break-system-packages numpy matplotlib pandas pillow pyjanitor pywavelets roifile scipy scikit-image scikit-learn shapely==2.0.7
# Replace __init__ with empty file to remove the default imports (to packages we don't have)
RUN :> /usr/lib/python3.12/site-packages/cellphe/__init__.py
# Add stub dependencies for packages that aren't needed for feature extraction
# NB: This is a hacky fix for not being able to just install the packages
# required for CellPhe's core functionality. It would be nice if CellPhe
# by default only depended on packaged for feature extraction, but
# optionally could also install those needed for segmentation/tracking
RUN mkdir /usr/lib/python3.12/site-packages/scyjava
RUN touch /usr/lib/python3.12/site-packages/scyjava/__init__.py
RUN mkdir /usr/lib/python3.12/site-packages/imagej
RUN touch /usr/lib/python3.12/site-packages/imagej/__init__.py
RUN mkdir /usr/lib/python3.12/site-packages/cellpose
RUN touch /usr/lib/python3.12/site-packages/cellpose/__init__.py
RUN touch /usr/lib/python3.12/site-packages/cellpose/models.py

# Copy python dependencies over from builder
FROM alpine:3.22.2 AS app

RUN apk add --no-cache \
      bash \
      python3=3.12.12-r0 \
      geos-dev

COPY --from=builder /usr/lib/python3.12 /usr/lib/python3.12
ENTRYPOINT ["/bin/bash"]

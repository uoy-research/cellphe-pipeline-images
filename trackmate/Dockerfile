ARG JDK_VERSION=openjdk21
ARG PYTHON_VERSION=3.12.12-r0
ARG PYTHON_VERSION_MINOR=python3.12
ARG MAVEN_VERSION=3.9.11
ARG SCYJAVA_VERSION=1.12.1
ARG PYIMAGEJ_VERSION=1.7.0

FROM alpine:3.22.2 AS builder

ARG JDK_VERSION
ARG PYTHON_VERSION
ARG PYTHON_VERSION_MINOR
ARG MAVEN_VERSION
ARG SCYJAVA_VERSION
ARG PYIMAGEJ_VERSION

# Build dependencies
RUN apk add --no-cache build-base $JDK_VERSION-jdk curl python3=$PYTHON_VERSION python3-dev linux-headers zip

# Fix JVM linkage issue
# See https://gitlab.alpinelinux.org/alpine/aports/-/issues/16606#top
ENV LD_LIBRARY_PATH="/usr/lib/jvm/java-21-openjdk/lib/server"

WORKDIR /app

# Install Maven
ENV MAVEN_HOME=/opt/maven
ENV PATH=$MAVEN_HOME/bin:$PATH
ENV MAVEN_VERSION=3.9.11
RUN curl https://dlcdn.apache.org/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz -o /tmp/maven.tar.gz && \
    tar xvf /tmp/maven.tar.gz -C /opt && \
    mv /opt/apache-maven-$MAVEN_VERSION $MAVEN_HOME

# Install Python dependencies
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
 && mv /root/.local/bin/uv /usr/local/bin/uv
RUN uv pip install --system --break-system-packages pyimagej==$PYIMAGEJ_VERSION scyjava==$SCYJAVA_VERSION

# Download Fiji + deps
COPY setup_imagej.py .
RUN uv run setup_imagej.py

# Application image with Python & Java runtime
FROM alpine:3.22.2 AS app

# Receive global vars
ARG JDK_VERSION
ARG PYTHON_VERSION
ARG PYTHON_VERSION_MINOR
ARG MAVEN_VERSION
ARG SCYJAVA_VERSION
ARG PYIMAGEJ_VERSION

RUN apk add --no-cache $JDK_VERSION-jre python3=$PYTHON_VERSION bash

# Fix JVM linkage issue
# See https://gitlab.alpinelinux.org/alpine/aports/-/issues/16606#top
ENV LD_LIBRARY_PATH="/usr/lib/jvm/java-21-openjdk/lib/server"

WORKDIR /app

# Copy python dependencies over from builder
COPY --from=builder /usr/lib/$PYTHON_VERSION_MINOR /usr/lib/$PYTHON_VERSION_MINOR

# Copy maven, maven cache, and jgo cache over
# NB: Setting HOME to /tmp to allow for Apptainer compatibility
COPY --from=builder /opt/maven /opt/maven
COPY --from=builder /root/.m2 /trackmate_libs/.m2
COPY --from=builder /root/.jgo /trackmate_libs/.jgo
ENV PATH=/opt/maven/bin:$PATH
ENV HOME=/trackmate_libs

ENTRYPOINT ["/bin/bash"]

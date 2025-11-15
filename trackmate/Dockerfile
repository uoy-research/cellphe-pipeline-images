FROM alpine:3.22.2 AS builder

# Build dependencies
RUN apk add --no-cache build-base openjdk21-jdk curl python3=3.12.12-r0 python3-dev linux-headers zip

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
RUN uv pip install --system --break-system-packages pyimagej scyjava

# Download Fiji + deps
COPY setup_imagej.py .
RUN uv run setup_imagej.py

# Application image with Python & Java runtime
FROM alpine:3.22.2 AS app
RUN apk add --no-cache openjdk21-jdk python3=3.12.12-r0

# Fix JVM linkage issue
# See https://gitlab.alpinelinux.org/alpine/aports/-/issues/16606#top
ENV LD_LIBRARY_PATH="/usr/lib/jvm/java-21-openjdk/lib/server"

WORKDIR /app

# Copy python dependencies over from builder
COPY --from=builder /usr/lib/python3.12 /usr/lib/python3.12

# Copy maven and maven cache over
ENV PATH=/opt/maven/bin:$PATH
COPY --from=builder /opt/maven /opt/maven
COPY --from=builder /root/.m2 /root/.m2

ENTRYPOINT ["/bin/bash"]

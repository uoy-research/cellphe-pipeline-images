FROM python:3.12-slim AS builder

# Build dependencies
RUN apt update && apt install -y \
    build-essential \
    default-jdk \
    curl \
    && rm -rf /var/lib/apt/lists/*

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

COPY setup_imagej.py .
RUN uv run setup_imagej.py

# Application image with Python & Java runtime
FROM python:3.12-slim AS app
RUN apt-get update && apt-get install -y \
    default-jdk \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy python dependencies over from builder
COPY --from=builder /usr/local/lib/python3.12 /usr/local/lib/python3.12

# Copy maven and maven cache over
ENV PATH=/opt/maven/bin:$PATH
COPY --from=builder /opt/maven /opt/maven
COPY --from=builder /root/.m2 /root/.m2

ENTRYPOINT ["/bin/bash"]

FROM python:3.13.9-slim-trixie

# Install R and system dependencies
RUN apt update && apt install -y r-base curl && rm -rf /var/cache/apt/archives /var/lib/apt/lists/*

# Install R packages
RUN R -e "install.packages(c('tidyverse', 'knitr', 'rmarkdown', 'reticulate'), repos='https://cloud.r-project.org/')"

# Install Python packages - uv first
COPY --from=ghcr.io/astral-sh/uv:0.9.8 /uv /uvx /bin/
RUN mkdir /app
COPY pyproject.toml /app
WORKDIR /app
RUN uv sync

# Install Quarto
RUN QUARTO_VERSION=1.8.25 && \
    curl -LO -k "https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.deb" && \
    dpkg -i "quarto-${QUARTO_VERSION}-linux-amd64.deb" && \
    rm -rf "quarto-${QUARTO_VERSION}-linux-amd64.deb"

ENTRYPOINT ["bash"]

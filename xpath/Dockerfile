FROM perl:5.43.4-slim-trixie

# Install dependencies
RUN apt update && apt install -y build-essential libexpat1-dev && rm -rf /var/cache/apt/archives /var/lib/apt/lists/*

# Install cpanm so can install cpan packages without configuring environment
RUN curl -L https://cpanmin.us | perl - App::cpanminus

# Install xpath module
RUN cpanm XML::XPath

ENTRYPOINT ["bash"]

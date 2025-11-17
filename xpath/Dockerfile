# ---- Stage 1: Build the XPath module ----
FROM alpine:3.22.2 AS builder

# Install Perl, build tools, and cpanminus
RUN apk add --no-cache perl perl-dev perl-app-cpanminus make gcc musl-dev expat-dev

# Install XML::XPath (provides the `xpath` CLI)
RUN cpanm --notest XML::XPath && rm -rf ~/.cpanm

# ---- Stage 2: Minimal runtime ----
FROM alpine:3.22.2

# Install bash + Perl runtime + XML library (no build tools)
RUN apk add --no-cache bash perl expat-dev

# Copy Perl modules + xpath executable from builder
COPY --from=builder /usr/local/lib/perl5 /usr/local/lib/perl5
COPY --from=builder /usr/local/share/perl5 /usr/local/share/perl5
COPY --from=builder /usr/local/bin/xpath /usr/local/bin/xpath

ENTRYPOINT ["/bin/bash"]

ARG PERL_VERSION=5.40.3-r0
ARG PERL_MAJOR_VERSION=perl5
ARG CPANMINUS_VERSION=1.7048-r0

# ---- Stage 1: Build the XPath module ----
FROM alpine:3.22.2 AS builder
ARG PERL_VERSION
ARG CPANMINUS_VERSION

# Install Perl, build tools, and cpanminus
RUN apk add --no-cache perl=$PERL_VERSION perl-dev perl-app-cpanminus=$CPANMINUS_VERSION make gcc musl-dev expat-dev

# Install XML::XPath (provides the `xpath` CLI)
RUN cpanm --notest XML::XPath && rm -rf ~/.cpanm

# ---- Stage 2: Minimal runtime ----
FROM alpine:3.22.2
ARG PERL_VERSION
ARG PERL_MAJOR_VERSION

# Install bash + Perl runtime + XML library (no build tools)
RUN apk add --no-cache bash perl=$PERL_VERSION expat-dev

# Copy Perl modules + xpath executable from builder
COPY --from=builder /usr/local/lib/$PERL_MAJOR_VERSION /usr/local/lib/$PERL_MAJOR_VERSION
COPY --from=builder /usr/local/share/$PERL_MAJOR_VERSION /usr/local/share/$PERL_MAJOR_VERSION
COPY --from=builder /usr/local/bin/xpath /usr/local/bin/xpath

ENTRYPOINT ["/bin/bash"]

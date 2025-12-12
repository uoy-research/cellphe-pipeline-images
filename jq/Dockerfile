FROM alpine:3.22.2 AS builder

# Install bash & jq
RUN apk add --no-cache \
    bash \
    jq=1.8.1-r0

FROM alpine:3.22.2 AS app
COPY --from=builder /bin/bash /bin/bash
COPY --from=builder /usr/bin/jq /usr/bin/jq
COPY --from=builder /usr/lib /usr/lib

ENTRYPOINT ["/bin/bash"]

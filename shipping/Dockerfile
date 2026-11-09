# syntax=docker/dockerfile:1

ARG RUST_VERSION=1.76
ARG APP_NAME=shipping

################################################################################
# Build stage
################################################################################

FROM rust:1.85.0 AS build

ARG APP_NAME

WORKDIR /app

# Install build dependencies for Debian
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        clang \
        lld \
        git \
        build-essential \
        pkg-config \
        libssl-dev && \
    rm -rf /var/lib/apt/lists/*

# Build the Rust application
COPY Cargo.toml Cargo.lock ./
COPY src ./src

RUN cargo build --locked --release && \
    cp "./target/release/${APP_NAME}" /bin/server

################################################################################
# Runtime stage
################################################################################

FROM debian:bookworm-slim AS final

# Install runtime libraries commonly required by Rust applications
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        libssl3 && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user
ARG UID=10001

RUN useradd \
    --uid "${UID}" \
    --no-create-home \
    --shell /usr/sbin/nologin \
    appuser

# Copy the application executable
COPY --from=build /bin/server /bin/server

# Run without root privileges
USER appuser

EXPOSE 7077

CMD ["/bin/server"]
# Build Stage
FROM rust:slim-bookworm as builder
WORKDIR /app
RUN apt-get update && apt-get install -y pkg-config libssl-dev && rm -rf /var/lib/apt/lists/*
COPY . .
ENV SQLX_OFFLINE=true
RUN cargo build --release

# Run Stage
FROM debian:bookworm-slim
WORKDIR /app
RUN apt-get update && apt-get install -y libssl3 ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/target/release/backend ./backend
EXPOSE 3000
CMD ["./backend"]

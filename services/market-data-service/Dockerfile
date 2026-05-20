# Multi-stage build for Market Data Service
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Install build dependencies
RUN apk add --no-cache git gcc musl-dev


# Copy source code
COPY . .
RUN go mod tidy
# Build binary
RUN CGO_ENABLED=1 GOOS=linux go build -a -installsuffix cgo -o market-data-service .

# Runtime stage
FROM alpine:3.18

WORKDIR /app

# Install runtime dependencies
RUN apk add --no-cache ca-certificates tzdata

# Add non-root user
RUN addgroup -g 1000 marketdata && adduser -D -u 1000 -G marketdata marketdata

# Copy binary from builder
COPY --from=builder /app/market-data-service .

# Set ownership
RUN chown -R marketdata:marketdata /app

# Switch to non-root user
USER marketdata

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

# Run application
ENTRYPOINT ["./market-data-service"]

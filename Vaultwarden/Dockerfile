# Neue Änderungen
FROM caddy:builder AS caddy-build
RUN xcaddy build \
    --with github.com/caddy-dns/cloudflare

# Stage 2: Combine Caddy and Vaultwarden into a single image
FROM vaultwarden/server:latest

# Copy the custom Caddy binary from the builder stage
COPY --from=caddy-build /usr/bin/caddy /usr/bin/caddy

# --- Environment Configuration ---
ENV ROCKET_PORT=8080
ENV ROCKET_ADDRESS=127.0.0.1
ENV XDG_DATA_HOME=/data/caddy_data
ENV XDG_CONFIG_HOME=/data/caddy_config

# --- Caddyfile Configuration ---
RUN mkdir -p /etc/caddy && cat > /etc/caddy/Caddyfile <<'EOF'
{$DOMAIN::443} {
    reverse_proxy 127.0.0.1:8080 {
        header_up X-Real-IP {remote_host}
    }
}
EOF

# --- Boot Script ---
RUN cat > /boot.sh <<'EOF'
#!/bin/sh
echo "Starting Caddy..."
caddy start --config /etc/caddy/Caddyfile
echo "Starting Vaultwarden..."
exec /start.sh
EOF

# Strip Windows line endings (CRLF -> LF) so Linux can read it, then make executable
RUN sed -i -e 's/\r$//' /boot.sh /etc/caddy/Caddyfile && chmod +x /boot.sh

EXPOSE 80 443

CMD ["/boot.sh"]

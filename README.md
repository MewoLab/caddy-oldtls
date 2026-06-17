A caddy fork that allow you to enable legacy/insecure TLS ciphersuites, for backward compatibility with legacy systems.

> [!WARNING]
> Caddy is aparently too modern to even allow the option to enable backward compatibility with older systems... 🥹  
> So... think again if you were considering migrating to caddy

<img width="1347" height="471" alt="image" src="https://github.com/user-attachments/assets/f2db2cda-96b7-4abe-972a-af305b39e3b7" />

## How to use (in docker)

Dockerfile:

```Dockerfile
FROM caddy:builder AS builder
RUN xcaddy build \
    --with github.com/mholt/caddy-l4 \
    --with github.com/caddyserver/caddy/v2=github.com/MewoLab/caddy-oldtls/v2@master

FROM caddy:alpine
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

compose.yml:

```compose.yml
services:
  caddy:
    build: .
    restart: unless-stopped
    environment:
      - GODEBUG=tlsrsakex=1
      - CADDY_TLS_ALLOW_INSECURE_CIPHER_SUITES=1
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./etc/caddy:/etc/caddy
      - ./data:/data
      - ./config:/config
```

etc/caddy/CaddyFile:

```CaddyFile
{
    debug
    default_sni example.com
}

https://example.com {
    tls {
        key_type rsa2048
        ciphers TLS_RSA_WITH_AES_128_GCM_SHA256 TLS_RSA_WITH_AES_256_GCM_SHA384 TLS_RSA_WITH_AES_128_CBC_SHA TLS_RSA_WITH_AES_256_CBC_SHA
    }

	respond 200 "OK"
}
```

then:

```sh
docker compose up -d
```

# Using a Reverse Proxy with HedgeDoc

If you want to use a reverse proxy to serve HedgeDoc, here are the essential
configs that you'll have to do.

This documentation will cover HTTPS setup, with comments for HTTP setup.

## HedgeDoc config

[Full explaination of the configuration options](../configuration.md)

| `config.json` parameter | Environment variable | Value | Example |
|-------------------------|----------------------|-------|---------|
| `domain` | `CMD_DOMAIN` | The full domain where your instance will be available | `hedgedoc.example.com` |
| `host` | `CMD_HOST` | An ip or domain name that is only available to HedgeDoc and your reverse proxy | `localhost` |
| `port` | `CMD_PORT` | An available port number on that IP | `3000` |
| `path` | `CMD_PATH` | path to UNIX domain socket to listen on (if specified, `host` or `CMD_HOST` and `port` or `CMD_PORT` are ignored) | `/var/run/hedgedoc.sock` |
| `protocolUseSSL` | `CMD_PROTOCOL_USESSL` | `true` if you want to serve your instance over SSL (HTTPS), `false` if you want to use plain HTTP | `true` |
| `useSSL` |  | `false`, the communications between HedgeDoc and the proxy are unencrypted | `false` |
| `urlAddPort` | `CMD_URL_ADDPORT` | `false`, HedgeDoc should not append its port to the URLs it links | `false` |
| `hsts.enable` | `CMD_HSTS_ENABLE` | `true` if you host over SSL, `false` otherwise | `true` |


## Reverse Proxy config

### Generic

The reverse proxy must allow websocket `Upgrade` requests at path `/sockets.io/`.

It must pass through the scheme used by the client (http or https).

### Nginx

Here is an example configuration for Nginx.

```
map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
}
server {
        server_name hedgedoc.example.com;

        location / {
                proxy_pass http://127.0.0.1:3000;
                proxy_set_header Host $host; 
                proxy_set_header X-Real-IP $remote_addr; 
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
                proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /socket.io/ {
                proxy_pass http://127.0.0.1:3000;
                proxy_set_header Host $host; 
                proxy_set_header X-Real-IP $remote_addr; 
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
        }

    listen [::]:443 ssl http2;
    listen 443 ssl http2;
    ssl_certificate fullchain.pem;
    ssl_certificate_key privkey.pem;
    include options-ssl-nginx.conf;
    ssl_dhparam ssl-dhparams.pem;
}
```

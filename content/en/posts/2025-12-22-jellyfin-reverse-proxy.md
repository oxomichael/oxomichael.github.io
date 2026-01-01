---
title: "Setting up Jellyfin with Chromecast using Docker and an Nginx Reverse Proxy"
date: "2025-12-22T10:00:00+02:00"
draft: false
translationKey: "jellyfin-reverse-proxy"
---

This guide explains how to install [Jellyfin](https://jellyfin.org/), a free software media server, using Docker, and how to make it securely accessible from the internet with an Nginx reverse proxy. We will also cover the specific configuration required to ensure Chromecast works correctly.

### Prerequisites

*   A server with Docker and Docker Compose installed.
*   A domain name (or a subdomain) pointing to your server's IP address.
*   Nginx installed on the same server.

### 1. Installing Jellyfin with Docker

The easiest way to deploy Jellyfin is by using Docker Compose. Create a `docker-compose.yml` file with the following content:

```yaml
version: "3.5"
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    user: 1000:1000 # UID:GID of the user, adapt if necessary
    network_mode: "host" # Required for Chromecast auto-discovery
    volumes:
      - ./config:/config # Jellyfin configuration storage
      - ./cache:/cache   # Cache files
      - /path/to/your/media:/media   # Your media library
    restart: "unless-stopped"
```

**Important points:**
*   **`user: 1000:1000`**: Replace with the UID and GID of the user who owns your media files to avoid permission issues. You can find this information with the `id <username>` command.
*   **`network_mode: "host"`**: This mode is crucial for Jellyfin to discover and communicate with Chromecast devices on your local network.
*   **Volumes**: Adjust the paths. `./config` and `./cache` will be created in the current directory. `/path/to/your/media` must point to your library.

Launch the container with the command:
```bash
docker-compose up -d
```

Jellyfin should now be accessible at `http://YOUR_LOCAL_IP:8096`.

### 2. Nginx Reverse Proxy Configuration

To securely access Jellyfin via your domain name (e.g., `jellyfin.yourdomain.com`), we will use Nginx as a reverse proxy.

#### Virtual Host Configuration

Create a configuration file for your site in Nginx (e.g., `/etc/nginx/conf.d/jellyfin.conf`).

```nginx
# /etc/nginx/conf.d/jellyfin.conf

# HTTP to HTTPS redirection
server {
    listen 80;
    listen [::]:80;
    server_name jellyfin.yourdomain.com;

    # Required for Let's Encrypt certificate renewal
    location ~ /.well-known/acme-challenge {
        allow all;
        root /var/www/html; # Adjust this path
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

# HTTPS configuration
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name jellyfin.yourdomain.com;

    # The default `client_max_body_size` is 1M, this might not be enough for some posters, etc.
    client_max_body_size 20M;

    # Paths to your SSL certificates (e.g., Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/jellyfin.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jellyfin.yourdomain.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/jellyfin.yourdomain.com/chain.pem;

    # Recommended SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_prefer_server_ciphers off;

    # Variable for the Jellyfin address
    set $jellyfin 127.0.0.1; # Local IP of the Docker server

    # Security / XSS Mitigation Headers
    # NOTE: X-Frame-Options may cause issues with the webOS app
    #add_header X-Frame-Options "SAMEORIGIN";
    #add_header X-XSS-Protection "0"; # Do NOT enable. This is obsolete/dangerous
    add_header X-Content-Type-Options "nosniff";

    # COOP/COEP. Disable if you use external plugins/images/assets
    add_header Cross-Origin-Opener-Policy "same-origin" always;
    add_header Cross-Origin-Embedder-Policy "require-corp" always;
    add_header Cross-Origin-Resource-Policy "same-origin" always;

    # Permissions policy. May cause issues on some clients
    add_header Permissions-Policy "accelerometer=(), ambient-light-sensor=(), battery=(), bluetooth=(), camera=(), clipboard-read=(), display-capture=(), document-domain=(), encrypted-media=(), gamepad=(),
geolocation=(), gyroscope=(), hid=(), idle-detection=(), interest-cohort=(), keyboard-map=(), local-fonts=(), magnetometer=(), microphone=(), payment=(), publickey-credentials-get=(), serial=(), sync-xhr=()
, usb=(), xr-spatial-tracking=()" always;

    # Content Security Policy
    # See: https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
    # Enforces https content and restricts JS/CSS to origin
    # External Javascript (such as cast_sender.js for Chromecast) must be whitelisted.
    # NOTE: The default CSP headers may cause issues with the webOS app
    #add_header Content-Security-Policy "default-src https: data: blob: ; img-src 'self' https://* ; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' https://www.gstatic.com https://www.youtube.com blob:; worker-src 'self' blob:; connect-src 'self'; object-src 'none'; font-src 'self'";
    # ===>
    add_header Content-Security-Policy "default-src https: data: blob: http://image.tmdb.org; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' https://www.gstatic.com/cv/js/sender/v1/cast_sender.js https://www.gstatic.com/eureka/clank/140/cast_sender.js https://www.gstatic.com/eureka/clank/141/cast_sender.js https://www.gstatic.com/eureka/clank/cast_sender.js https://www.youtube.com blob:; worker-src 'self' blob:; connect-src 'self'; object-src 'none'; frame-ancestors 'self'";

    # Redirect to web interface
    location = / {
        return 302 https://$host/web/;
    }

    # Proxy main Jellyfin traffic
    location / {
        proxy_pass http://$jellyfin:8096;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;

        # Disable buffering when the nginx proxy gets very resource heavy upon streaming
        proxy_buffering off;
    }

    # location block for /web - This is purely for aesthetics so /web/#!/ works instead of having to go to /web/index.html/#!/
    location = /web/ {
        # Proxy main Jellyfin traffic
        proxy_pass http://$jellyfin:8096/web/index.html;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;
    }

    # Proxy Jellyfin Websockets traffic
    location /socket {
         proxy_pass http://$jellyfin:8096;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;
    }

    # Cache Video Streams
    location ~* ^/Videos/(.*)/(?!live) {
        # Set size of a slice (this amount will be always requested from the backend by nginx)
        # Higher value means more latency, lower more overhead
        # This size is independent of the size clients/browsers can request
        slice 2m;

        proxy_cache jellyfin-videos;
        proxy_cache_valid 200 206 301 302 30d;
        proxy_ignore_headers Expires Cache-Control Set-Cookie X-Accel-Expires;
        proxy_cache_use_stale error timeout invalid_header updating http_500 http_502 http_503 http_504;
        proxy_connect_timeout 15s;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        # Transmit slice range to the backend
        proxy_set_header Range $slice_range;

        # This saves bandwidth between the proxy and jellyfin, as a file is only downloaded one time instead of multiple times when multiple clients want to at the same time
        # The first client will trigger the download, the other clients will have to wait until the slice is cached
        # Esp. practical during SyncPlay
        proxy_cache_lock on;
        proxy_cache_lock_age 60s;

        proxy_pass http://$jellyfin:8096;
        proxy_cache_key "jellyvideo$uri?MediaSourceId=$arg_MediaSourceId&VideoCodec=$arg_VideoCodec&AudioCodec=$arg_AudioCodec&AudioStreamIndex=$arg_AudioStreamIndex&VideoBitrate=$arg_VideoBitrate&AudioBitrate=$arg_AudioBitrate&SubtitleMethod=$arg_SubtitleMethod&TranscodingMaxAudioChannels=$arg_TranscodingMaxAudioChannels&RequireAvc=$arg_RequireAvc&SegmentContainer=$arg_SegmentContainer&MinSegments=$arg_MinSegments&BreakOnNonKeyFrames=$arg_BreakOnNonKeyFrames&h264-profile=$h264Profile&h264-level=$h264Level&slicerange=$slice_range";

        # add_header X-Cache-Status $upstream_cache_status; # This is only for debugging cache
    }

    # Cache images (inside server block)
    location ~ /Items/(.*)/Images {
        proxy_pass http://$jellyfin:8096;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;

        proxy_cache jellyfin;
        proxy_cache_revalidate on;
        proxy_cache_lock on;
        # add_header X-Cache-Status $upstream_cache_status; # This is only to check if cache is working
    }

    # Downloads limit (inside server block)
    location ~ /Items/(.*)/Download$ {
        proxy_pass http://$jellyfin:8096;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;

        limit_rate 5000k; # Speed limit (here is on kb/s)
        limit_conn addr 1; # Number of simultaneous downloads per IP
        limit_conn_status 460; # Custom error handling
        # proxy_buffering on; # Be sure buffering is on (it is by default on nginx), otherwise limits won't work
    }
}
```

Adding video cache management on the proxy server

```nginx
# /etc/nginx/conf.d/jellyfin-server-http.conf

# --- Cache Video Streams
# Must be in HTTP block
# Set in-memory cache-metadata size in keys_zone, size of video caching and how many days a cached object should persist
proxy_cache_path  /var/cache/nginx/jellyfin-videos levels=1:2 keys_zone=jellyfin-videos:100m inactive=90d max_size=35000m;
map $request_uri $h264Level { ~(h264-level=)(.+?)& $2; }
map $request_uri $h264Profile { ~(h264-profile=)(.+?)& $2; }

# --- Cache Images
proxy_cache_path /var/cache/nginx/jellyfin levels=1:2 keys_zone=jellyfin:100m max_size=15g inactive=30d use_temp_path=off;

# --- Rate Limit Downloads
limit_conn_zone $binary_remote_addr zone=addr:10m;
```


#### Activating the Configuration

```bash
# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### 3. Making Chromecast Functional

The main challenge with Chromecast over an HTTPS reverse proxy is the browser's Content Security Policy (CSP). By default, Jellyfin's policy is too restrictive and blocks the Google Cast scripts.

The Nginx configuration above overrides this header to allow the necessary scripts from `gstatic.com`.

**Key CSP Directive:**
```nginx
add_header Content-Security-Policy "default-src https: data: blob: http://image.tmdb.org; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' https://www.gstatic.com/cv/js/sender/v1/cast_sender.js https://www.gstatic.com/eureka/clank/140/cast_sender.js https://www.gstatic.com/eureka/clank/141/cast_sender.js https://www.gstatic.com/eureka/clank/cast_sender.js https://www.youtube.com blob:; worker-src 'self' blob:; connect-src 'self'; object-src 'none'; frame-ancestors 'self'";
```
This rule allows the loading of `cast_sender.js` scripts and enables the browser to communicate with your Chromecast devices.

Note: By checking the browser's HTTP console, it is possible to see the allowed requests and thus validate the configuration.

### Conclusion

Your Jellyfin instance is now securely accessible via HTTPS and is compatible with Chromecast. Remember to complete the initial Jellyfin setup through its web interface to add your media libraries.

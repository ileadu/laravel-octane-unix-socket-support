# Laravel Octane – UNIX Socket Support for Swoole

Add UNIX Domain Socket (UDS) support to [Laravel Octane](https://github.com/laravel/octane) when using the Swoole server.
This enables high-performance, local-only communication between Octane and Nginx (or other reverse proxies), ideal for Docker or Linux server deployments.

## ✨ Features

- ✅ Supports UNIX Domain Socket (UDS) via `--host=/tmp/octane.sock`
- ✅ Automatically detects socket paths and binds Swoole accordingly
- ✅ Fully backward-compatible (TCP continues to work if UDS is not used)
- ✅ Optimized for Linux and Docker environments
- 🚫 Ignored on Windows (no UDS support)

## 📦 Installation

Due to Composer limitations, patches defined inside packages like this one are **not automatically applied**.
To activate the UNIX socket support, you must manually define the patch in your Laravel project's `composer.json`.

Add the following to the `extra.patches` section in your root `composer.json`:

```json
"extra": {
  "laravel": {
    "dont-discover": []
  },
  "patches": {
    "laravel/octane": {
      "Add UNIX socket support to Octane": "vendor/ileadu/laravel-octane-unix-socket-support/patches/octane-unix-socket.patch"
    }
  }
}
```

To apply this patch to Laravel Octane v2.10+

```bash
composer require ileadu/laravel-octane-unix-socket-support
```

## 💡 About This Patch

This package applies the patch from [laravel/octane#1032](https://github.com/laravel/octane/pull/1032) and enhances the existing `--host` option to support UNIX socket paths

## 🚀 Usage

Start Octane with Swoole and a UNIX socket:

```bash
php artisan octane:start --server=swoole --host=/tmp/octane.sock
```

If the `--host` value is a valid file path (e.g. starts with `/` and ends with `.sock`),
Octane will automatically use a UNIX domain socket instead of TCP.

## 🔧 Nginx Integration Example

Example nginx.conf:

```nginx
location /index.php {
    try_files /not_exists @octane;
}

location / {
    try_files $uri $uri/ @octane;
}

location @octane {
    proxy_http_version 1.1;
    proxy_set_header Host $http_host;
    proxy_set_header Scheme $scheme;
    proxy_set_header SERVER_PORT $server_port;
    proxy_set_header REMOTE_ADDR $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    proxy_pass http://unix:/tmp/octane.sock:$request_uri;
}
```

## 💡 Why Use UNIX Domain Sockets?

🔥 Faster than TCP for **local** communication (less overhead)

🔒 More secure – no external port exposure

🐳 Perfect for Docker or systemd socket-activation setups

🧩 Clean and easy integration with Nginx via proxy_pass

## ❓ Fallback Behavior

If `--host` is a UNIX socket path → Swoole binds to the socket.

If `--host` is a hostname or IP → Octane behaves normally (TCP mode).

Patch is ignored on Windows platforms.

## 📝 Patch Details

This package includes a patch against Laravel Octane v2.10.0, enhancing the `--host` option to support UNIX socket binding for Swoole.

## 📄 License

MIT

## 🙏 Credits

Thanks to the Laravel and Octane teams for building an incredible high-performance foundation.
This patch is a community-contributed improvement for specific deployment scenarios.

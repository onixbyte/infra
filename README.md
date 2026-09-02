# infra

The infrastructure repository of onixbyte, hosting the CI workflows used to build and maintain the runtime environment.

## Directory structure

```
infra/
└── .github/workflows/
    └── build-caddy.yaml   # GitHub Actions workflow that builds a customised Caddy binary
```

## Building a customised Caddy

The `build-caddy.yaml` workflow uses [xcaddy](https://github.com/caddyserver/xcaddy) to compile a specified version of Caddy with plugins injected, and uploads the compiled binary as an artefact. The workflow can only be triggered manually.

### How to trigger the build

1. Open the **Actions** page of the repository.
2. Select the **Build Customised Caddy** workflow from the left-hand panel.
3. Click **Run workflow** and fill in the parameters (see the table below).
4. Wait for the build to finish, then download the `caddy-custom-linux-amd64` artefact from the build run page.

### Input parameters

| Parameter | Description | Required | Default |
|-----------|-------------|:---:|---------|
| `caddy_version` | Caddy version to build, e.g. `latest`, `v2.8.4` | Yes | `latest` |
| `caddy_plugin` | Comma-separated plugin module paths, e.g. `github.com/caddy-dns/cloudflare, github.com/caddyserver/nginx-adapter`. Leave empty to build vanilla Caddy. | No | — |

### Plugin input behaviour

The table below shows how the value of `caddy_plugin` maps to the actual `xcaddy` command (using the default `latest` version):

| Input (`caddy_plugin`) | Command that runs |
|------------------------|-------------------|
| `github.com/caddy-dns/cloudflare, github.com/caddyserver/nginx-adapter` | `xcaddy build latest --with github.com/caddy-dns/cloudflare --with github.com/caddyserver/nginx-adapter` |
| `github.com/caddy-dns/cloudflare,github.com/caddyserver/nginx-adapter` | Same as above (spaces are optional) |
| `github.com/caddy-dns/cloudflare` | `xcaddy build latest --with github.com/caddy-dns/cloudflare` |
| *(empty)* | `xcaddy build latest` (vanilla Caddy) |

> Empty entries are skipped automatically, so `plugin1, , plugin2` behaves the same as `plugin1, plugin2`.

## Building locally

To reproduce the same build on your own machine you need Go 1.22+:

```sh
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest

# Compile a specific version with a plugin injected
xcaddy build v2.8.4 --with github.com/caddy-dns/cloudflare

# Inject several plugins at once
xcaddy build latest \
  --with github.com/caddy-dns/cloudflare \
  --with github.com/caddyserver/nginx-adapter
```

When the build finishes, an executable named `caddy` is produced in the current directory.

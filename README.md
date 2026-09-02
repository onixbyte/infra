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

### Uploading to S3-compatible storage

When the `S3_BUCKET` variable is set, the compiled binary is also uploaded to any S3-compatible object storage as `caddy-<version>-linux-amd64`. Leaving `S3_BUCKET` unset skips this step.

Configure the repository settings (**Settings → Secrets and variables → Actions**):

| Kind | Name | Description |
|------|------|-------------|
| Variable | `S3_ENDPOINT` | Endpoint of your S3-compatible service, e.g. `https://tos-s3-cn-hongkong.volces.com` (Volcano TOS), `https://s3.amazonaws.com`, or a MinIO/R2 endpoint |
| Variable | `S3_REGION` | S3 region used for signing, e.g. `cn-hongkong`, `us-east-1` (defaults to `cn-hongkong` if unset) |
| Variable | `S3_BUCKET` | Bucket to upload to. Leave unset to skip the upload |
| Secret | `S3_ACCESS_KEY` | AccessKey ID |
| Secret | `S3_SECRET_KEY` | AccessKey Secret |

The upload uses the AWS CLI with the endpoint from `S3_ENDPOINT`, VirtualHostStyle addressing and SigV4 signing. The key needs write permission on the bucket. Note that a few providers (e.g. MinIO in path-style mode) require `path` addressing instead of `virtual` — edit the `aws configure set default.s3.addressing_style` line in the workflow accordingly.

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

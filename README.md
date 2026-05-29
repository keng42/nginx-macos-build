# nginx for macOS arm64

Build a native NGINX binary for Apple Silicon macOS using GitHub Actions.

This repository produces a standalone nginx-macos-arm64 binary intended for local reverse-proxy and HTTPS gateway use on macOS without installing Homebrew or Docker Desktop.

## Build target

- Platform: macOS arm64 / Apple Silicon
- GitHub Actions runner: macos-15
- Runtime dependency goal: macOS system libraries only
- Intended runtime mode: non-root user process listening on high ports such as 8443

## Included components

| Component | Version |
| --------- | ------- |
| nginx     | 1.30.2  |
| OpenSSL   | 3.5.6   |
| PCRE2     | 10.47   |
| zlib      | 1.3.2   |

All source archives are downloaded from the upstream GitHub Releases pages and verified against pinned SHA-256 checksums before compilation.

## Enabled functionality

The build includes commonly useful HTTP gateway functionality:

- HTTPS / TLS support
- HTTP/2 support
- Reverse proxy support
- gzip support
- PCRE2 regular expressions with JIT
- stub_status status endpoint support

No third-party NGINX modules are included.

## Build

Run the workflow manually:

1. Open the repository on GitHub.
2. Go to Actions.
3. Select Build nginx for macOS arm64.
4. Click Run workflow.
5. After the job succeeds, download the generated release assets.

The release contains:

```
nginx-macos-arm64
SHA256SUMS
nginx-<version>-macos-arm64.tar.gz
```

The tarball also contains the bundled third-party license files.

## Install on macOS

Create a user-owned runtime directory:

```bash
BASE="$HOME/Library/Application Support/nginx"
mkdir -p \
  "$BASE/bin" \
  "$BASE/conf" \
  "$BASE/logs" \
  "$BASE/run" \
  "$BASE/certs" \
  "$BASE/tmp"
```

Verify the downloaded binary before installing it:

```bash
cd ~/Downloads shasum -a 256 -c SHA256SUMS
```

Install the binary:

```bash
BASE="$HOME/Library/Application Support/nginx"
cp ~/Downloads/nginx-macos-arm64 "$BASE/bin/nginx"
chmod 700 "$BASE/bin/nginx"
```

Because the binary is downloaded from GitHub Releases and is not notarized by Apple, macOS may attach a quarantine attribute. Remove it only after verifying the SHA-256 checksum:

```bash
xattr -d com.apple.quarantine "$HOME/Library/Application Support/nginx/bin/nginx" 2>/dev/null || true
```

Check the binary:

```bash
"$HOME/Library/Application Support/nginx/bin/nginx" -V
```

## Run

Test the configuration:

```bash
BASE="$HOME/Library/Application Support/nginx"
"$BASE/bin/nginx" -p "$BASE/" -c conf/nginx.conf -t
```

Start NGINX:

```bash
BASE="$HOME/Library/Application Support/nginx"
"$BASE/bin/nginx" -p "$BASE/" -c conf/nginx.conf
```

Reload configuration:

```bash
BASE="$HOME/Library/Application Support/nginx"
"$BASE/bin/nginx" -p "$BASE/" -c conf/nginx.conf -s reload
```

Stop gracefully:

```bash
BASE="$HOME/Library/Application Support/nginx"
"$BASE/bin/nginx" -p "$BASE/" -c conf/nginx.conf -s quit
```

## Security notes

- This binary is intended to run as a normal macOS user and should not listen directly on ports 80 or 443.
- Keep private TLS keys readable only by the account running NGINX.
- Do not store a private root CA key in the runtime service directory.
- Do not expose local services directly to the public Internet unless that exposure is intentional and independently secured.
- Rebuild the binary when nginx or OpenSSL publishes relevant security fixes.
- Never disable source checksum verification in the workflow.

## Updating

To upgrade any bundled component:

1. Update its pinned version in the workflow.
2. Replace its SHA-256 checksum with the verified upstream checksum.
3. Run the GitHub Actions workflow.
4. Download and verify the new release artifact.
5. Replace the local binary.
6. Test the configuration.
7. Reload or restart NGINX.

## Upstream projects

- NGINX: https://github.com/nginx/nginx
- OpenSSL: https://github.com/openssl/openssl
- PCRE2: https://github.com/PCRE2Project/pcre2
- zlib: https://github.com/madler/zlib

## License

This repository contains build automation only.

The produced binary includes software from NGINX, OpenSSL, PCRE2, and zlib. Refer to the license files bundled in the release archive for their respective terms.

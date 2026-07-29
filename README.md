# dogecoin-core

Parameterized Dogecoin Core Docker image and Helm chart for Kubernetes.

## Docker

Single parameterized Dockerfile supporting any Dogecoin Core version. Multi-arch: `linux/amd64`, `linux/arm64`.

**GPG:** `KEYS` build-arg is **required** (space-separated fingerprints). The image build:

1. Imports only those fingerprints from keyservers
2. Requires a **GOODSIG** on upstream `SHA256SUMS.asc` (expired `EXPKEYSIG` alone fails)
3. Requires **VALIDSIG** fingerprint ∈ the `KEYS` allowlist

There is **no** TOFU / auto-import-from-signature path.

```bash
# Build locally
docker buildx build \
  --build-arg VERSION=$(cat VERSION) \
  --build-arg "KEYS=$(tr '\n' ' ' < KEYS)" \
  --platform linux/amd64 \
  -t dogecoin-core:$(cat VERSION) \
  docker/
```

Image: `ghcr.io/kub0-ai/dogecoin-core`

## Helm Chart

StatefulSet-based deployment with persistent storage for blockchain data.

Defaults:

- **RPC allowlist** is private RFC1918 ranges only (`10/8`, `172.16/12`, `192.168/16`) — not `0.0.0.0/0`. Override `config` for your CNI CIDRs.
- **securityContext:** `runAsNonRoot`, uid/gid `101`, `allowPrivilegeEscalation: false`, drop `ALL` capabilities.
- Service type **ClusterIP** (RPC not published externally by default).

```bash
helm install dogecoin ./chart \
  --set persistence.storageClass=block-sata \
  --set persistence.size=100Gi
```

## Version Management

- `VERSION` — current Dogecoin Core version (single source of truth)
- `KEYS` — GPG signing key fingerprints for binary verification (allowlist)
- `watch-release.yml` — daily cron (and manual dispatch) checks `dogecoin/dogecoin` for newer max-semver releases. When a bump is needed it:
  1. Creates/updates branch `release/<version>` with the new `VERSION`
  2. Opens a **pull request** with a human **GPG checklist** (verify keys / update `KEYS` / review notes)
  3. **Does not** push `main` directly
  4. Merging the PR to `main` (paths: `VERSION`) triggers `build.yml` image rebuild
- `backfill.yml` — manual dispatch to build a specific historical version; **KEYS must be non-empty** (workflow input or repo `KEYS` file). Empty keys fail closed — no TOFU rebuild path.

## License

MIT

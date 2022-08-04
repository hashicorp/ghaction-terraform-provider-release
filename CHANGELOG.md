# v2.1.0

community: Use `crazy-max/ghaction-import-gpg@v5` instead of deprecated `hashicorp/ghaction-import-gpg@v2.1.0`
community: Update to `actions/checkout@v3`, `actions/download-artifact@v3`, `actions/setup-go@v3`, and `goreleaser/goreleaser-action@v3`
hashicorp: Run on GitHub Actions Custom Runners with `[custom, linux, large]`
hashicorp: Update to `actions/checkout@v3`, `actions/download-artifact@v3`, `actions/setup-go@v3`, and `goreleaser/goreleaser-action@v3`

# v2.0.0

hashicorp: Use `hashicorp/actions-hc-releases-create-metadata` and `hashicorp/actions-hc-releases-promote` instead of `hc-releases` directly
hashicorp: Update to `hashicorp/setup-hc-releases@v2`

# v1.1.0

hashicorp: Add `hc-releases-aws-role-session-duration` input to configure IAM Role session duration, in seconds, for `hc-releases`. Defaults to previous value of `3600` (1 hour).

# v1.0.1

hashicorp: Checkout using `fetch-depth: 0` to prevent commit-driven workflows from failing to fetch the associated tag

# v1.0.0

Initial release

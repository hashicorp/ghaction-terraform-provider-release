## 3.0.0 (January 19, 2024)

BREAKING CHANGES:

* all: Workflows are now using `actions/download-artifact@v4` which is incompatible with `actions/upload-artifact@v3`. Calling workflows must be upgraded to `actions/upload-artifact@v4` when including release notes ([#76](https://github.com/hashicorp/ghaction-terraform-provider-release/issues/76))

NOTES:

* hashicorp: Prevent NodeJS 12 deprecation warnings by upgrading hashicorp/setup-signore ([#60](https://github.com/hashicorp/ghaction-terraform-provider-release/issues/60))

## 2.3.0 (February 28, 2023)

NOTES:

* community: Calls to other actions are now version pinned by Git commit rather than tag ([#31](https://github.com/hashicorp/ghaction-terraform-provider-release/issues/31))
* hashicorp: Calls to other actions are now version pinned by Git commit rather than tag ([#31](https://github.com/hashicorp/ghaction-terraform-provider-release/issues/31))
* community: Replace deprecated `goreleaser --rm-dist` flag with new `--clean` flag ([#39](https://github.com/hashicorp/ghaction-terraform-provider-release/issues/39))
* hashicorp: Replace deprecated `goreleaser --rm-dist` flag with new `--clean` flag ([#39](https://github.com/hashicorp/ghaction-terraform-provider-release/issues/39))

ENHANCEMENTS:

* hashicorp: Support self-hosted runners without sudo support ([#42](https://github.com/hashicorp/ghaction-terraform-provider-release/issues/42))

# v2.2.0

ENHANCEMENTS:

* community: Added `setup-go-version-file` input for specifying a file path which contains the Go version, such as `.go-version` or `go.mod` ([#25](https://github.com/hashicorp/ghaction-terraform-provider-release/pull/25))
* community: Added `git-ref` input for specifying the Git reference for checkout ([#26](https://github.com/hashicorp/ghaction-terraform-provider-release/pull/26))
* hashicorp: Added `setup-go-version-file` input for specifying a file path which contains the Go version, such as `.go-version` or `go.mod` ([#25](https://github.com/hashicorp/ghaction-terraform-provider-release/pull/25))
* hashicorp: Use `product-version` input for specifying the Git reference for checkout, which enables releases from non-default branches ([#26](https://github.com/hashicorp/ghaction-terraform-provider-release/pull/26))


# v2.1.0

* community: Use `crazy-max/ghaction-import-gpg@v5` instead of deprecated `hashicorp/ghaction-import-gpg@v2.1.0`
* community: Update to `actions/checkout@v3`, `actions/download-artifact@v3`, `actions/setup-go@v3`, and `goreleaser/goreleaser-action@v3`
* hashicorp: Run on GitHub Actions Custom Runners with `[custom, linux, large]`
* hashicorp: Update to `actions/checkout@v3`, `actions/download-artifact@v3`, `actions/setup-go@v3`, and `goreleaser/goreleaser-action@v3`

# v2.0.0

* hashicorp: Use `hashicorp/actions-hc-releases-create-metadata` and `hashicorp/actions-hc-releases-promote` instead of `hc-releases` directly
* hashicorp: Update to `hashicorp/setup-hc-releases@v2`

# v1.1.0

* hashicorp: Add `hc-releases-aws-role-session-duration` input to configure IAM Role session duration, in seconds, for `hc-releases`. Defaults to previous value of `3600` (1 hour).

# v1.0.1

* hashicorp: Checkout using `fetch-depth: 0` to prevent commit-driven workflows from failing to fetch the associated tag

# v1.0.0

Initial release

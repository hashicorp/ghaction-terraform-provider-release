# ghaction-terraform-provider-release

[Reusable GitHub Action Workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) for releasing HashiCorp, partner, and community Terraform Providers into the [public Terraform Registry](https://registry.terraform.io/). These workflows can be called directly or used as examples for implementing a custom workflow.

## Usage

### Partner and Community Providers

This workflow automatically handles:

- Checking out the Git repository using a Git reference in the `git-ref` input, or the `HEAD` of the default branch.
- Installing the [Go](https://go.dev/) version specified in the `setup-go-version` or `setup-go-version-file` input.
- Importing the GPG key for signing specified in the `gpg-private-key` secret input.
- Downloading and using the `release-notes` artifact, if the `release-notes` input is enabled.
- Running [Goreleaser](https://goreleaser.com/), which builds the provider for multiple platforms, creates a GitHub Release, and uploads release assets.

Ensure all instructions have been followed on the [Publishing Providers page](https://developer.hashicorp.com/terraform/registry/providers/publishing), including setting up GPG keys in the [Terraform Registry](https://registry.terraform.io/).

To use the reusable workflow, update the `.github/workflows/release.yml` configuration to include the following:

```yaml
jobs:
  # ... potentially other jobs ...
  terraform-provider-release:
    name: 'Terraform Provider Release'
    uses: hashicorp/ghaction-terraform-provider-release/.github/workflows/community.yml@v4
    secrets:
      gpg-private-key: '${{ secrets.GPG_PRIVATE_KEY }}'
    with:
      setup-go-version-file: 'go.mod'
```

See the [workflow file](https://github.com/hashicorp/ghaction-terraform-provider-release/blob/main/.github/workflows/community.yml) for all available inputs.

### HashiCorp Providers

This workflow automatically handles:

- Checking out the Git repository using the `product-version` input as the Git reference.
- Installing the [Go](https://go.dev/) version specified in the `setup-go-version` or `setup-go-version-file` input.
- Installing and authenticating `signore` and `hc-releases` for signing and publishing releasing assets.
- Downloading and using the `release-notes` artifact, if the `release-notes` input is enabled.
- Running [Goreleaser](https://goreleaser.com/), which builds the provider for multiple platforms, creates a GitHub Release, and uploads release assets.

To use the reusable workflow, update the `.github/workflows/release.yml` configuration to include the following:

```yaml
jobs:
  # ... potentially other jobs ...
  terraform-provider-release:
    name: 'Terraform Provider Release'
    uses: hashicorp/ghaction-terraform-provider-release/.github/workflows/hashicorp.yml@v4
    secrets:

      hc-releases-github-token: '${{ secrets.HASHI_RELEASES_GITHUB_TOKEN }}'
      hc-releases-host-staging: '${{ secrets.HC_RELEASES_HOST_STAGING }}'
      hc-releases-host-prod: '${{ secrets.HC_RELEASES_HOST_PROD }}'
      hc-releases-key-prod: '${{ secrets.HC_RELEASES_KEY_PROD }}'
      hc-releases-key-staging: '${{ secrets.HC_RELEASES_KEY_STAGING }}'
      hc-releases-terraform-registry-sync-token: '${{ secrets.TF_PROVIDER_RELEASE_TERRAFORM_REGISTRY_SYNC_TOKEN }}'
      setup-signore-github-token: '${{ secrets.HASHI_SIGNORE_GITHUB_TOKEN }}'
      signore-client-id: '${{ secrets.SIGNORE_CLIENT_ID }}'
      signore-client-secret: '${{ secrets.SIGNORE_CLIENT_SECRET }}'
    with:
      setup-go-version-file: 'go.mod'
      product-version: 'v1.2.3'
```

See the [workflow file](https://github.com/hashicorp/ghaction-terraform-provider-release/blob/main/.github/workflows/hashicorp.yml) for all available inputs.

## Additional Features

Each of the reusable workflows can be used with additional automation features. These configurations are an example of many possibilities.

### Setting Release Notes From Changelog

Changelog contents can be used as the release description in GitHub. The changelog file must be fully prepared before executing the release job, so it has the expected contents. For example, if using the provider development [Changelog Specification](https://www.terraform.io/docs/extend/best-practices/versioning.html#changelog-specification) (including the date is optional), a job can be configured to get the changelog contents between the last version and the current version being released:

```yaml
jobs:
  # ... other jobs ...
  release-notes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Generate Release Notes
        run: sed -n -e "1{/# /d;}" -e "2{/^$/d;}" -e "/# $(git describe --abbrev=0 --exclude="$(git describe --abbrev=0 --match='v*.*.*' --tags)" --match='v*.*.*' --tags | tr -d v)/q;p" CHANGELOG.md > release-notes.txt
      - uses: actions/upload-artifact@v4
        with:
          name: release-notes
          path: release-notes.txt
          retention-days: 1
```

To pass the release notes contents between jobs, a GitHub Actions artifact named `release-notes` with the `release-notes.txt` file must be uploaded. Then, the release job can be configured to include those release notes:

```yaml
jobs:
  # ... other jobs ...
  terraform-provider-release:
    name: 'Terraform Provider Release'
    needs: [release-notes]
    uses: hashicorp/ghaction-terraform-provider-release/.github/workflows/community.yml@v4
    secrets:
      gpg-private-key: '${{ secrets.GPG_PRIVATE_KEY }}'
    with:
      release-notes: true
      setup-go-version-file: 'go.mod'
```

## Releases

These reusable workflows are [semantically versioned](https://semver.org/) and tagged using conventional GitHub Actions versioning practices:

- `v#.#.#`: Stable release tags. See also this project's [CHANGELOG.md](./CHANGELOG.md).
- `v#`: Updated release tag that contains the latest stable v#.#.# matching the major version.

## Contributing

### License Headers
All source code files (excluding autogenerated files like `go.mod`, prose, and files excluded in [.copywrite.hcl](.copywrite.hcl)) must have a license header at the top.

This can be autogenerated by installing the HashiCorp [`copywrite`](https://github.com/hashicorp/copywrite#getting-started) tool and running `copywrite headers` in the root of the repository.

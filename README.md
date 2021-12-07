# ghaction-terraform-provider-release

[Reusable GitHub Action Workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) for releasing HashiCorp, partner, and community Terraform Providers. These workflows can be called directly or used as examples for implementing a custom workflow.

## Usage

_If you are using the [terraform-provider-scaffolding template repository](https://github.com/hashicorp/terraform-provider-scaffolding), the `.github/workflow/release.yml` file will require modifications from the original template files._

### Partner and Community Providers

This workflow automatically handles:

- Installing the [Go](https://go.dev/) version specified in the `setup-go-version` input.
- Importing the GPG key for signing specified in the `gpg-private-key` secret input.
- Downloading and using the `release-notes` artifact, if the `release-notes` input is enabled.
- Running [Goreleaser](https://goreleaser.com/), which builds the provider for multiple platforms, creates a GitHub Release, and uploads release assets.

Ensure all instructions have been followed on the [Publishing Providers page](https://www.terraform.io/docs/registry/providers/publishing.html), including setting up GPG keys in the [Terraform Registry](https://registry.terraform.io/).

To use the reusable workflow, update the `.github/workflows/release.yml` configuration to include the following:

```yaml
jobs:
  # ... potentially other jobs ...
  terraform-provider-release:
    name: 'Terraform Provider Release'
    uses: bflad/ghaction-terraform-provider-release/.github/workflows/community.yml@v1
    secrets:
      gpg-private-key: '${{ secrets.GPG_PRIVATE_KEY }}'
    with:
      setup-go-version: '1.17.x'
```

See the [workflow file](https://github.com/bflad/ghaction-terraform-provider-release/blob/main/.github/workflows/community.yml) for all available inputs.

### HashiCorp Providers

This workflow automatically handles:

- Installing the [Go](https://go.dev/) version specified in the `setup-go-version` input.
- Installing and authenticating `signore` and `hc-releases` for signing and publishing releasing assets.
- Downloading and using the `release-notes` artifact, if the `release-notes` input is enabled.
- Running [Goreleaser](https://goreleaser.com/), which builds the provider for multiple platforms, creates a GitHub Release, and uploads release assets.

To use the reusable workflow, update the `.github/workflows/release.yml` configuration to include the following:

```yaml
jobs:
  # ... potentially other jobs ...
  terraform-provider-release:
    name: 'Terraform Provider Release'
    uses: bflad/ghaction-terraform-provider-release/.github/workflows/hashicorp.yml@v1
    secrets:
      hc-releases-aws-access-key-id: '${{ secrets.TF_PROVIDER_RELEASE_AWS_ACCESS_KEY_ID }}'
      hc-releases-aws-secret-access-key: '${{ secrets.TF_PROVIDER_RELEASE_AWS_SECRET_ACCESS_KEY }}'
      hc-releases-aws-role-arn: '${{ secrets.TF_PROVIDER_RELEASE_AWS_ROLE_ARN }}'
      hc-releases-fastly-api-token: '${{ secrets.FASTLY_STATIC_PURGE_TOKEN }}'
      hc-releases-github-token: '${{ secrets.HC_RELEASES_TOKEN }}'
      hc-releases-terraform-registry-sync-token: '${{ secrets.TF_PROVIDER_RELEASE_TERRAFORM_REGISTRY_SYNC_TOKEN }}'
      setup-signore-github-token: '${{ secrets.SETUP_SIGNORE_GITHUB_TOKEN }}'
      signore-client-id: '${{ secrets.SIGNORE_CLIENT_ID }}'
      signore-client-secret: '${{ secrets.SIGNORE_CLIENT_SECRET }}'
    with:
      setup-go-version: '1.17.x'
```

See the [workflow file](https://github.com/bflad/ghaction-terraform-provider-release/blob/main/.github/workflows/hashicorp.yml) for all available inputs.

## Additional Features

Each of the reusable workflows can be used with additional automation features. These configurations are an example of many possibilities.

### Reading Go Version From File

Until [actions/setup-go](https://github.com/actions/setup-go/) supports [fetching the version from a file](https://github.com/actions/setup-go/issues/23), a separate job can be used to read a file containing the Go version, such as:

```yaml
jobs:
  # ... other jobs ...
  go-version:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.go-version.outputs.version }}
    steps:
      - uses: actions/checkout@v2
      - id: go-version
        run: echo "::set-output name=version::$(cat ./.go-version)"
```

Then the output of that job can be passed as an input to the release job:

```yaml
jobs:
  # ... other jobs ...
  terraform-provider-release:
    name: 'Terraform Provider Release'
    needs: [go-version]
    uses: bflad/ghaction-terraform-provider-release/.github/workflows/community.yml@v1
    secrets:
      gpg-private-key: '${{ secrets.GPG_PRIVATE_KEY }}'
    with:
      setup-go-version: '${{ needs.go-version.outputs.version }}'
```

### Setting Release Notes From Changelog

Changelog contents can be used as the release description in GitHub. The changelog file must be fully prepared before executing the release job, so it has the expected contents. For example, if using the provider development [Changelog Specification](https://www.terraform.io/docs/extend/best-practices/versioning.html#changelog-specification) (including the date is optional), a job can be configured to get the changelog contents between the last version and the current version being released:

```yaml
jobs:
  # ... other jobs ...
  release-notes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Generate Release Notes
        run: sed -n -e "1{/# /d;}" -e "2{/^$/d;}" -e "/# $(git describe --abbrev=0 --exclude="$(git describe --abbrev=0 --match='v*.*.*' --tags)" --match='v*.*.*' --tags | tr -d v)/q;p" CHANGELOG.md > release-notes.txt
      - uses: actions/upload-artifact@v2
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
    uses: bflad/ghaction-terraform-provider-release/.github/workflows/community.yml@v1
    secrets:
      gpg-private-key: '${{ secrets.GPG_PRIVATE_KEY }}'
    with:
      release-notes: true
      setup-go-version: '1.17.x'
```

## Releases

These reusable workflows are [semantically versioned](https://semver.org/) and tagged using conventional GitHub Actions versioning practices:

- `v#.#.#`: Stable release tags. See also this project's [CHANGELOG.md](./CHANGELOG.md).
- `v#`: Updated release tag that contains the latest stable v#.#.# matching the major version.

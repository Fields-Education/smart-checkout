# Smart Checkout Action

A GitHub Action that wraps `actions/checkout` with automatic fetch-depth
calculation based on commits since the last release.

## Why?

- `fetch-depth: 0` (full clone) is slow for large repositories
- `fetch-depth: 1` (shallow) breaks tools that need git history (cocogitto,
  changelogs, moon affected detection)
- This action calculates the minimum depth needed: commits since last release

## Usage

```yaml
- uses: fields-education/smart-checkout@v1
```

Drop-in replacement for `actions/checkout` - all inputs are supported.

## Inputs

All inputs from [`actions/checkout@v6`](https://github.com/actions/checkout) are
supported and passed through directly.

### Smart Checkout Specific

| Input         | Description                                                 | Default |
| ------------- | ----------------------------------------------------------- | ------- |
| `fetch-depth` | `auto` (calculate from last release), `0` (full), or number | `auto`  |

### Repository

| Input        | Description                                     | Default             |
| ------------ | ----------------------------------------------- | ------------------- |
| `repository` | Repository name with owner (e.g., `owner/repo`) | `github.repository` |
| `ref`        | Branch, tag, or SHA to checkout                 | triggering ref      |
| `token`      | PAT or GITHUB_TOKEN for fetching                | `github.token`      |
| `path`       | Relative path under $GITHUB_WORKSPACE           | repository root     |

### SSH Authentication

| Input             | Description                         | Default |
| ----------------- | ----------------------------------- | ------- |
| `ssh-key`         | SSH key for fetching the repository |         |
| `ssh-known-hosts` | Additional known hosts entries      |         |
| `ssh-strict`      | Perform strict host key checking    | `true`  |
| `ssh-user`        | User for SSH connection             | `git`   |

### Credentials

| Input                 | Description                                 | Default |
| --------------------- | ------------------------------------------- | ------- |
| `persist-credentials` | Configure token/SSH key in local git config | `true`  |

### Clone Options

| Input                       | Description                                                 | Default |
| --------------------------- | ----------------------------------------------------------- | ------- |
| `clean`                     | Run `git clean -ffdx && git reset --hard HEAD` before fetch | `true`  |
| `filter`                    | Partial clone filter (overrides sparse-checkout)            |         |
| `sparse-checkout`           | Sparse checkout patterns (newline separated)                |         |
| `sparse-checkout-cone-mode` | Use cone-mode for sparse checkout                           | `true`  |

### Fetch Options

| Input           | Description                                        | Default |
| --------------- | -------------------------------------------------- | ------- |
| `fetch-tags`    | Fetch tags even if fetch-depth > 0                 | `true`  |
| `show-progress` | Show progress output when fetching                 | `true`  |
| `lfs`           | Download Git-LFS files                             | `false` |
| `submodules`    | Checkout submodules (`true`, `false`, `recursive`) | `false` |

### Other

| Input                | Description                                | Default |
| -------------------- | ------------------------------------------ | ------- |
| `set-safe-directory` | Add repo path to Git global safe.directory | `true`  |
| `github-server-url`  | Base URL for GitHub instance               |         |

## Outputs

### Smart Checkout Specific

| Output                  | Description                          |
| ----------------------- | ------------------------------------ |
| `fetch-depth`           | The depth that was actually used     |
| `commits-since-release` | Number of commits since last release |
| `last-release-tag`      | The release tag used for calculation |

### Pass-through from actions/checkout

| Output   | Description                                 |
| -------- | ------------------------------------------- |
| `ref`    | The branch, tag or SHA that was checked out |
| `commit` | The commit SHA that was checked out         |

## Behavior

1. **Auto mode** (default): Queries GitHub API for commits since last release
2. **No releases**: Falls back to full clone (`depth=0`)
3. **API error**: Falls back to full clone with warning
4. **Explicit depth**: Uses provided value directly

## Examples

### Basic usage

```yaml
steps:
  - uses: fields-education/smart-checkout@v1
    id: checkout

  - run: |
      echo "Depth used: ${{ steps.checkout.outputs.fetch-depth }}"
      echo "Commits since release: ${{ steps.checkout.outputs.commits-since-release }}"
      echo "Last release: ${{ steps.checkout.outputs.last-release-tag }}"
```

### Force full clone

```yaml
- uses: fields-education/smart-checkout@v1
  with:
    fetch-depth: "0"
```

### Disable credential persistence

```yaml
- uses: fields-education/smart-checkout@v1
  with:
    persist-credentials: false
```

### PR workflow with head SHA

```yaml
on: pull_request

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: fields-education/smart-checkout@v1
        with:
          ref: ${{ github.event.pull_request.head.sha }}
```

### With SSH key

```yaml
- uses: fields-education/smart-checkout@v1
  with:
    ssh-key: ${{ secrets.DEPLOY_KEY }}
```

### Sparse checkout

```yaml
- uses: fields-education/smart-checkout@v1
  with:
    sparse-checkout: |
      src/
      tests/
```
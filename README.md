# Ubuntu FIPS Updates Action

A reusable GitHub Actions workflow for running workloads in an Ubuntu Pro VM with FIPS-updates enabled.

## Overview

This workflow:
1. Launches an Ubuntu 24.04 LXD VM
2. Attaches an Ubuntu Pro subscription
3. Enables FIPS-updates
4. Reboots to activate FIPS
5. Runs your custom workload
6. Detaches from Ubuntu Pro

## Usage

Call the reusable workflow from your own workflow:

```yaml
name: "Test FIPS"
on:
  push:
  pull_request:
  workflow_dispatch:

jobs:
  test-fips:
    uses: <owner>/<repo>/.github/workflows/enable-pro.yml@main
    with:
      workload: 'your-commands-here'
    secrets:
      UBUNTU_PRO_TOKEN: ${{ secrets.UBUNTU_PRO_TOKEN }}
```

## Inputs

| Name | Description | Required |
|------|-------------|----------|
| `workload` | Commands to run inside the VM after Pro FIPS-updates is enabled | Yes |

## Secrets

| Name | Description | Required |
|------|-------------|----------|
| `UBUNTU_PRO_TOKEN` | Your Ubuntu Pro subscription token | Yes |

More info at https://ubuntu.com/pro.

## Example

Run OpenSSL FIPS validation:

```yaml
jobs:
  validate-fips:
    uses: <owner>/<repo>/.github/workflows/enable-pro.yml@main
    with:
      workload: |
        openssl version
        cat /proc/sys/crypto/fips_enabled
        pro status --format json
    secrets:
      UBUNTU_PRO_TOKEN: ${{ secrets.UBUNTU_PRO_TOKEN }}
```

## Requirements

- GitHub-hosted runner (ubuntu-latest)
- Valid Ubuntu Pro token stored as a repository secret

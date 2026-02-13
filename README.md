# Ubuntu FIPS-Updates Action

[![CI](https://github.com/canonical/ubuntu-fips-updates-action/actions/workflows/ci.yml/badge.svg)](https://github.com/canonical/ubuntu-fips-updates-action/actions/workflows/ci.yml)

A GitHub Action that sets up an Ubuntu VM with FIPS-updates enabled via Ubuntu Pro. Use this to test your software in a FIPS-compliant environment.

## Quick Start

```yaml
name: Test in FIPS
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup FIPS environment
        uses: canonical/ubuntu-fips-updates-action@v1
        with:
          pro-token: ${{ secrets.PRO_TOKEN }}
          shared-path: ${{ github.workspace }}

      - name: Run tests in FIPS mode
        uses: canonical/ubuntu-fips-updates-action/run@v1
        with:
          working-directory: /shared
          commands: |
            make test

      - name: Cleanup
        if: always()
        uses: canonical/ubuntu-fips-updates-action/cleanup@v1
```

## How It Works

1. Sets up LXD on the GitHub runner
2. Launches an Ubuntu VM (24.04 by default)
3. Attaches your Ubuntu Pro subscription
4. Enables `fips-updates` and reboots
5. Verifies FIPS mode is active
6. Your commands run inside the FIPS-enabled VM

## Actions Reference

### Setup (`canonical/ubuntu-fips-updates-action`)

Creates and configures the FIPS-enabled VM.

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `pro-token` | Ubuntu Pro token | **Yes** | - |
| `vm-name` | Name for the VM | No | `fips-vm` |
| `ubuntu-version` | Ubuntu version | No | `24.04` |
| `cpu-limit` | VM CPU count | No | `2` |
| `memory-limit` | VM memory | No | `4GiB` |
| `shared-path` | Host path to mount at `/shared` | No | - |

| Output | Description |
|--------|-------------|
| `vm-name` | Name of the created VM |

### Run (`canonical/ubuntu-fips-updates-action/run`)

Executes commands inside the VM.

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `commands` | Commands to execute | **Yes** | - |
| `vm-name` | VM name | No | `fips-vm` |
| `working-directory` | Directory inside VM | No | - |
| `shell` | Shell to use | No | `bash` |

### Cleanup (`canonical/ubuntu-fips-updates-action/cleanup`)

Detaches from Ubuntu Pro and deletes the VM. **Always run this in an `if: always()` step.**

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `vm-name` | VM name | No | `fips-vm` |
| `detach-pro` | Detach from Pro first | No | `true` |

## Examples

### Basic FIPS Validation

```yaml
- uses: canonical/ubuntu-fips-updates-action@v1
  with:
    pro-token: ${{ secrets.PRO_TOKEN }}

- uses: canonical/ubuntu-fips-updates-action/run@v1
  with:
    commands: |
      cat /proc/sys/crypto/fips_enabled  # Should output: 1
      openssl version
      openssl list -providers

- if: always()
  uses: canonical/ubuntu-fips-updates-action/cleanup@v1
```

### Build and Test a Project

```yaml
- uses: actions/checkout@v4

- uses: canonical/ubuntu-fips-updates-action@v1
  with:
    pro-token: ${{ secrets.PRO_TOKEN }}
    shared-path: ${{ github.workspace }}

- uses: canonical/ubuntu-fips-updates-action/run@v1
  with:
    commands: apt-get update && apt-get install -y build-essential

- uses: canonical/ubuntu-fips-updates-action/run@v1
  with:
    working-directory: /shared
    commands: |
      ./configure
      make
      make test

- if: always()
  uses: canonical/ubuntu-fips-updates-action/cleanup@v1
```

### Retrieve Artifacts

```yaml
- uses: canonical/ubuntu-fips-updates-action/run@v1
  with:
    commands: ./run-tests.sh --output /tmp/results.xml

- name: Copy results from VM
  run: |
    mkdir -p results
    sudo lxc file pull fips-vm/tmp/results.xml results/

- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: results/
```

### Custom VM Configuration

```yaml
- uses: canonical/ubuntu-fips-updates-action@v1
  with:
    pro-token: ${{ secrets.PRO_TOKEN }}
    vm-name: my-test-vm
    ubuntu-version: '22.04'
    cpu-limit: '4'
    memory-limit: '8GiB'

- uses: canonical/ubuntu-fips-updates-action/run@v1
  with:
    vm-name: my-test-vm
    commands: echo "Running on Ubuntu 22.04 with 4 CPUs"

- if: always()
  uses: canonical/ubuntu-fips-updates-action/cleanup@v1
  with:
    vm-name: my-test-vm
```

## Requirements

- **Runner:** `ubuntu-latest` (GitHub-hosted)
- **Ubuntu Pro token:** Get one at [ubuntu.com/pro](https://ubuntu.com/pro)

Store your token as a repository secret named `PRO_TOKEN`.

## Supported Ubuntu Versions

| Version | FIPS-updates Support |
|---------|---------------------|
| 24.04 LTS | ✅ |
| 22.04 LTS | ✅ |
| 20.04 LTS | ✅ |

## License

[Apache License 2.0](LICENSE)

## Contributing

See the [examples](examples/) directory for sample workflows.

Issues and pull requests are welcome at [github.com/canonical/ubuntu-fips-updates-action](https://github.com/canonical/ubuntu-fips-updates-action).

# WireGuard Action

A GitHub Actions composite action that sets up a WireGuard VPN connection on Ubuntu runners.

## Usage

```yaml
- name: Connect to VPN
  uses: leukeleu/wireguard-connect@main
  with:
    WIREGUARD_CONFIG: ${{ secrets.WIREGUARD_CONFIG }}
```

## Inputs

| Input | Required | Description |
|---|---|---|
| `WIREGUARD_CONFIG` | Yes | Full WireGuard configuration file contents (store this as a repository or organization secret) |
| `VERIFY_HOSTNAME` | No | A hostname to test resolution against after the tunnel is up. Leave empty to skip this check. |

## How it works

1. Installs `wireguard` and `resolvconf` via `apt`
2. Writes the configuration to `/etc/wireguard/wg0.conf` (mode `600`)
3. Fetches GitHub's IP ranges from the public `api.github.com/meta` endpoint
4. Brings up the tunnel with `wg-quick up wg0`
5. Adds static routes for GitHub's IP ranges through the runner's original gateway, so the runner <-> GitHub connection survives the VPN's default-route takeover
6. Waits up to 15s for a WireGuard handshake, failing the step if none is established
7. If `VERIFY_HOSTNAME` is set, waits up to 10s for that hostname to resolve through `wg0`, failing the step if it cannot

## Requirements

- An Ubuntu runner (`ubuntu-*`)

## Example

1. Export a WireGuard configuration file from your VPN provider's dashboard
2. Add the file contents as a secret named `WIREGUARD_CONFIG` in your repository or organization settings
3. Add the action as a step before any steps that need VPN access:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Connect to VPN
        uses: leukeleu/wireguard-connect@main
        with:
          WIREGUARD_CONFIG: ${{ secrets.WIREGUARD_CONFIG }}
          VERIFY_HOSTNAME: internal.example.com

      - name: Deploy
        run: ./deploy.sh
```

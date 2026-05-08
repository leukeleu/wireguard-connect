# WireGuard Action

A GitHub Actions composite action that sets up a WireGuard VPN connection on Ubuntu runners.

## Usage

```yaml
- name: Connect to VPN
  uses: leukeleu/wireguard-action@main
  with:
    WIREGUARD_CONFIG: ${{ secrets.WIREGUARD_CONFIG }}
```

## Inputs

| Input | Required | Description |
|---|---|---|
| `WIREGUARD_CONFIG` | Yes | Full WireGuard configuration file contents (store this as a repository or organization secret) |

## How it works

1. Installs `wireguard` and `resolvconf` via `apt`
2. Writes the configuration to `/etc/wireguard/wg0.conf` (mode `600`)
3. Brings up the tunnel with `wg-quick up wg0`

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
        uses: leukeleu/wireguard-action@main
        with:
          WIREGUARD_CONFIG: ${{ secrets.WIREGUARD_CONFIG }}

      - name: Deploy
        run: ./deploy.sh
```

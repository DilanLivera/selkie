# selkie

A collection of tools for coding remotely.

## Tools

### `lab-connect` — Homelab SSH + Zellij

Automates SSH-ing into a homelab server over Tailscale and attaching to a Zellij session.

**Requirements:**

- Tailscale (authenticated)
- Zellij (on the remote server)

**Usage:**

```bash
cp lab-connect/lab-connect.conf.example ~/.config/selkie/lab-connect.conf
# Fill in SSH_USER and SSH_HOST

chmod +x lab-connect/lab-connect
./lab-connect/lab-connect
```

**Configuration:**

Config is loaded from the first path found:

1. `$SELKIE_CONFIG` environment variable
2. `~/.config/selkie/lab-connect.conf`
3. `./lab-connect.conf`

| Key               | Description                                    | Default |
| ----------------- | ---------------------------------------------- | ------- |
| `SSH_USER`        | Username on the remote server                  | —       |
| `SSH_HOST`        | Tailscale IP or hostname of the remote server  | —       |
| `SSH_RETRY_COUNT` | Number of SSH connection attempts before giving up | `3` |
| `SSH_RETRY_DELAY` | Seconds to wait between retry attempts         | `5`     |


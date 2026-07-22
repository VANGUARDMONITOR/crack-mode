# crack-mode 🔌

**Switch your Ubuntu system between GUI and headless crack mode on demand.**

When you're running long hashcat sessions on a hybrid laptop-server, every megabyte of RAM counts. `crack-mode` isolates into `multi-user.target` (no display manager, no desktop) to free resources, then restores your GUI when done.

Part of the [VANGUARDMONITOR/Trinity](https://github.com/VANGUARDMONITOR/Trinity) ecosystem.

---

## Why

Most password-cracking rigs are headless servers. But if your crack box is also your daily-driver laptop with GNOME/KDE, the desktop environment consumes RAM that could go to hashcat's mask buffers, rule chains, or simply leave more room for the OS page cache.

| Metric | With GUI | Headless | Gain |
|--------|----------|----------|------|
| Free RAM | ~8.5 GiB | ~9.2 GiB | **+700 MiB** |
| GPU MD5 | 11,214 MH/s | 11,541 MH/s | *noise (±3%)* |
| Network | stable | stable (re-affirmed) | ✅ |
| Systemd gateway | active | active (linger) | ✅ |

**The GPU performance doesn't improve** — the gain is RAM, not compute. This matters when cracking large wordlists, keeping big rule chains in memory, or running multiple hashcat instances.

---

## Requirements

- **Ubuntu** (or any Linux with `systemd` + `NetworkManager`)
- `bash` ≥ 4
- `sudo` access for `systemctl isolate`
- `nmcli` (NetworkManager CLI) — for WiFi re-affirmation after target switch
- Optional: `sensors` (lm-sensors) for temperature in `status`
- Optional: systemd user services with `loginctl enable-linger` if you run services (like a gateway) that must survive the GUI switch

---

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/VANGUARDMONITOR/crack-mode/main/crack-mode \
  -o ~/.local/bin/crack-mode
chmod +x ~/.local/bin/crack-mode
```

Or clone the repo:

```bash
git clone https://github.com/VANGUARDMONITOR/crack-mode.git
cp crack-mode/crack-mode ~/.local/bin/
chmod +x ~/.local/bin/crack-mode
```

---

## Usage

```bash
crack-mode on       # Switch to multi-user.target (kill GUI)
crack-mode off      # Switch back to graphical.target
crack-mode status   # Show current target, RAM, temperature, gateway health
crack-mode help     # Show this help
```

### Idempotent scenarios

| Current state | `crack-mode on` | `crack-mode off` |
|---------------|-----------------|------------------|
| GUI (graphical.target) | ✅ Switches to headless | ✅ Skips — "already in GUI mode" |
| Headless (multi-user.target) | ✅ Skips — "already in headless mode" | ✅ Switches to GUI |

---

## Configuration

| Environment variable | Default | Purpose |
|---------------------|---------|---------|
| `CRACKMODE_GATEWAY` | `openclaw-gateway.service` | Systemd user service to check after switch |
| `CRACKMODE_IFACE` | *(auto-detect)* | WiFi interface to re-affirm after isolate |

---

## Design notes

### The WiFi flap (and why the script re-affirms the network)

`systemctl isolate multi-user.target` kills the display manager, which can disturb `wpa_supplicant` / `NetworkManager` enough to trigger a WiFi disconnection/reconnection cycle. On the author's system this caused a ~6-second DNS outage (`EAI_AGAIN`).

The script mitigates this by:
1. Sleeping 5 seconds after the isolate
2. Re-affirming the WiFi connection via `nmcli device connect`
3. Verifying Internet reachability with `ping`

This was discovered empirically (and fixed in the same session) — see the commit history for `post-isolate network stability` if you're curious about the forensic details.

### Idempotence

Both `on` and `off` check the **current active target** (`systemctl list-units --type=target --state=active`) — not the default boot target — before executing. If you're already in the requested mode, the script prints "already in [mode]" and exits without calling `isolate`.

### Multiple active targets

On Ubuntu, `graphical.target` pulls in `multi-user.target` as a dependency, so both can appear active simultaneously. The script accounts for this: `off` checks for `graphical` (if present, you're in GUI mode), while `on` checks that `multi-user` is active **and** `graphical` is absent (true headless state).

---

## License

MIT — see [LICENSE](LICENSE).

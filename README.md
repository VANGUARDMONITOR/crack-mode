# crack-mode 🔌

[🇪🇸 Español](#español) | [🇺🇸 English](#english)

---

<a name="español"></a>
# 🇪🇸 crack-mode

**Swichea entre modo GUI y headless en Ubuntu para liberar RAM durante sesiones de cracking.**

Cuando tu servidor es también tu laptop del día a día con GNOME/KDE, el escritorio consume RAM que podrías usar en buffers de máscaras de hashcat, cadenas de reglas, o simplemente dejar más memoria para el caché del sistema operativo. `crack-mode` aísla a `multi-user.target` (sin gestor de pantalla, sin escritorio) para liberar recursos, y restaura la GUI cuando terminas.

Parte del ecosistema [VANGUARDMONITOR/Trinity](https://github.com/VANGUARDMONITOR/Trinity).

---

## ¿Por qué?

La mayoría de los rigs de cracking son servidores headless. Pero si tu máquina de cracking es también tu laptop personal, el entorno de escritorio consume RAM valiosa.

| Métrica | Con GUI | Sin GUI | Ganancia |
|---------|:-------:|:-------:|:--------:|
| RAM libre | ~8.5 GiB | ~9.2 GiB | **+700 MiB** |
| GPU MD5 | 11,214 MH/s | 11,541 MH/s | *ruido (±3%)* |
| Red | estable | estable (reafirmada) | ✅ |
| Gateway systemd | activo | activo (linger) | ✅ |

**El rendimiento de la GPU no mejora** — la ganancia es en RAM, no en cómputo. Esto importa cuando procesas wordlists grandes, mantienes cadenas de reglas extensas en memoria, o ejecutas múltiples instancias de hashcat.

---

## Requisitos

- **Ubuntu** (o cualquier Linux con `systemd` + `NetworkManager`)
- `bash` ≥ 4
- Acceso `sudo` para `systemctl isolate`
- `nmcli` (NetworkManager CLI) — para reafirmar WiFi tras el cambio de target
- Opcional: `sensors` (lm-sensors) para temperatura en `status`
- Opcional: servicios de usuario systemd con `loginctl enable-linger` si ejecutas servicios (como un gateway) que deben sobrevivir al apagado de la GUI

---

## Instalación

```bash
curl -fsSL https://raw.githubusercontent.com/VANGUARDMONITOR/crack-mode/main/crack-mode \
  -o ~/.local/bin/crack-mode
chmod +x ~/.local/bin/crack-mode
```

O clonando el repo:

```bash
git clone https://github.com/VANGUARDMONITOR/crack-mode.git
cp crack-mode/crack-mode ~/.local/bin/
chmod +x ~/.local/bin/crack-mode
```

---

## Uso

```bash
crack-mode on       # Cambiar a multi-user.target (apagar GUI)
crack-mode off      # Volver a graphical.target
crack-mode status   # Mostrar target actual, RAM, temperatura, estado del gateway
crack-mode help     # Mostrar ayuda
```

### Escenarios idempotentes

| Estado actual | `crack-mode on` | `crack-mode off` |
|---------------|-----------------|------------------|
| GUI (`graphical.target`) | ✅ Cambia a headless | ✅ Salta — "ya en modo GUI" |
| Headless (`multi-user.target`) | ✅ Salta — "ya en modo cracking" | ✅ Cambia a GUI |

---

## Configuración

| Variable de entorno | Valor por defecto | Propósito |
|--------------------|-------------------|-----------|
| `CRACKMODE_GATEWAY` | *(vacío = saltar)* | Servicio de usuario systemd a verificar tras el cambio (opcional) |
| `CRACKMODE_IFACE` | *(detección automática)* | Interfaz WiFi a reafirmar tras el isolate |

---

## Notas de diseño

### El flap de WiFi (y por qué el script reafirma la red)

`systemctl isolate multi-user.target` mata el gestor de pantalla, lo que puede perturbar `wpa_supplicant` / `NetworkManager` lo suficiente como para gatillar un ciclo de desconexión/reconexión WiFi. En el sistema del autor esto causó ~6 segundos de caída de DNS (`EAI_AGAIN`).

El script mitiga esto:
1. Esperando 5 segundos tras el isolate
2. Reafirmando la conexión WiFi vía `nmcli device connect`
3. Verificando conectividad a Internet con `ping`

Esto fue descubierto empíricamente (y corregido en la misma sesión) — ver el historial de commits para `post-isolate network stability`.

### Targets activos múltiples

En Ubuntu, `graphical.target` incluye a `multi-user.target` como dependencia, así que ambos pueden aparecer activos simultáneamente. El script lo maneja: `off` verifica si `graphical` está presente (si sí, estás en modo GUI), mientras que `on` verifica que `multi-user` esté activo **y** `graphical` esté ausente (headless real).

---

## Licencia

MIT — ver [LICENSE](LICENSE).

---

<a name="english"></a>
# 🇺🇸 English

**Switch between GUI and headless mode on Ubuntu to free RAM for hashcat cracking sessions.**

Part of the [VANGUARDMONITOR/Trinity](https://github.com/VANGUARDMONITOR/Trinity) ecosystem.

---

## Why

Most password-cracking rigs are headless servers. But if your crack box is also your daily-driver laptop with GNOME/KDE, the desktop environment consumes RAM that could go to hashcat's mask buffers, rule chains, or simply leave more room for the OS page cache.

| Metric | With GUI | Headless | Gain |
|--------|:-------:|:--------:|:----:|
| Free RAM | ~8.5 GiB | ~9.2 GiB | **+700 MiB** |
| GPU MD5 | 11,214 MH/s | 11,541 MH/s | *noise (±3%)* |
| Network | stable | stable (re-affirmed) | ✅ |
| Systemd gateway | active | active (linger) | ✅ |

**GPU performance does not improve** — the gain is RAM, not compute.

---

## Requirements

- **Ubuntu** (or any Linux with `systemd` + `NetworkManager`)
- `bash` ≥ 4
- `sudo` access for `systemctl isolate`
- `nmcli` (NetworkManager CLI)
- Optional: `sensors` (lm-sensors) for temperature in `status`
- Optional: systemd user services with `loginctl enable-linger`

---

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/VANGUARDMONITOR/crack-mode/main/crack-mode \
  -o ~/.local/bin/crack-mode
chmod +x ~/.local/bin/crack-mode
```

## Usage

```bash
crack-mode on       # Switch to multi-user.target (kill GUI)
crack-mode off      # Switch back to graphical.target
crack-mode status   # Show current target, RAM, temperature, gateway health
crack-mode help     # Show help
```

### Idempotent scenarios

| Current state | `crack-mode on` | `crack-mode off` |
|---------------|-----------------|------------------|
| GUI (`graphical.target`) | ✅ Switches to headless | ✅ Skips — "already in GUI mode" |
| Headless (`multi-user.target`) | ✅ Skips — "already in headless mode" | ✅ Switches to GUI |

---

## Configuration

| Environment variable | Default | Purpose |
|---------------------|---------|---------|
| `CRACKMODE_GATEWAY` | *(empty = skip)* | Systemd user service to check after switch (optional) |
| `CRACKMODE_IFACE` | *(auto-detect)* | WiFi interface to re-affirm after isolate |

---

## Design notes

### The WiFi flap (and why the script re-affirms the network)

`systemctl isolate multi-user.target` kills the display manager, which can disturb `wpa_supplicant` / `NetworkManager` enough to trigger a WiFi disconnection/reconnection cycle. On the author's system this caused a ~6-second DNS outage (`EAI_AGAIN`).

The script mitigates this by:
1. Sleeping 5 seconds after the isolate
2. Re-affirming the WiFi connection via `nmcli device connect`
3. Verifying Internet reachability with `ping`

### Multiple active targets

On Ubuntu, `graphical.target` pulls in `multi-user.target` as a dependency, so both can appear active simultaneously. `crack-mode` accounts for this: `off` checks for `graphical` (if present, you're in GUI mode), while `on` checks that `multi-user` is active **and** `graphical` is absent (true headless state).

---

## License

MIT — see [LICENSE](LICENSE).

---

*Maintained by [VANGUARDMONITOR](https://github.com/VANGUARDMONITOR) — part of the [Trinity](https://github.com/VANGUARDMONITOR/Trinity) ecosystem.*

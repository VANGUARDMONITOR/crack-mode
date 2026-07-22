# crack-mode 🔌

[🇪🇸 Español](#español) | [🇺🇸 English](#english)

<img src="docs/CSL_PURO.png" alt="CSL" width="32" align="right"/>
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
| `CRACKMODE_IFACE` | *(auto todas)* | Interfaz específica a reafirmar (override opcional) |

---

## Notas de diseño

### Re-afirmación de red agnóstica del medio

`systemctl isolate multi-user.target` puede perturbar `NetworkManager` lo suficiente como para gatillar reconexiones en cualquier interfaz — no solo WiFi. El caso crítico documentado fue WiFi (`wpa_supplicant` ligado a la sesión de usuario, ~6s de caída DNS), pero Ethernet y otros medios también pueden flapear.

El script re-afirma **todas las conexiones activas**:
1. Espera 5 segundos tras el isolate
2. Detecta conexiones activas via `nmcli -f NAME,DEVICE,TYPE connection show --active`
3. Re-afirma cada una: `nmcli device connect <interfaz>`
4. Verifica conectividad a Internet con `ping`

Si no hay conexiones activas, avisa y sigue. Si se define `CRACKMODE_IFACE`, solo re-afirma esa.

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
| `CRACKMODE_IFACE` | *(auto-detect all)* | Specific interface to re-affirm (optional override) |

---

## Design notes

### Medium-agnostic network re-affirmation

`systemctl isolate multi-user.target` can disturb NetworkManager enough to trigger reconnections on any interface — not just WiFi. The documented critical case was WiFi (`wpa_supplicant` tied to the user session, ~6s DNS outage), but Ethernet and other media can flap too.

The script re-affirms **all active connections**:
1. Sleeps 5 seconds after the isolate
2. Detects active connections via `nmcli -f NAME,DEVICE,TYPE connection show --active`
3. Re-affirms each one: `nmcli device connect <interface>`
4. Verifies Internet reachability with `ping`

If no active connections are found, it warns and continues. If `CRACKMODE_IFACE` is set, only that interface is re-affirmed.

### Multiple active targets

On Ubuntu, `graphical.target` pulls in `multi-user.target` as a dependency, so both can appear active simultaneously. `crack-mode` accounts for this: `off` checks for `graphical` (if present, you're in GUI mode), while `on` checks that `multi-user` is active **and** `graphical` is absent (true headless state).

---

## License

MIT — see [LICENSE](LICENSE).

---

*Maintained by [VANGUARDMONITOR](https://github.com/VANGUARDMONITOR) — part of the [Trinity](https://github.com/VANGUARDMONITOR/Trinity) ecosystem.*

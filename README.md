<div align="left">

![logo](img/icon.png)

# ttydBridge

A lightweight Docker-based bridge that exposes the host terminal through a browser by using **ttyd**.

This project is designed for environments where you intentionally want direct access to the host shell from the web UI. Instead of keeping the terminal isolated inside the container, **ttydBridge** connects into the host namespace and makes the host terminal available through the browser.

</div>

## What it does

- Exposes the **host terminal** in the browser
- Supports **system login** or a custom startup command
- Can optionally enable **HTTP basic authentication**
- Can optionally enable **HTTPS / SSL**
- Supports **IPv6**
- Can automatically open the selected port on the host via **iptables**
- Supports terminal **theme customization** via JSON
- Supports additional **custom ttyd arguments**
- Tracks the started ttyd process with a **PID file** for cleaner shutdown handling

## Important note

This container is intentionally designed to bypass normal container isolation.
It uses host namespaces and privileged access so the browser terminal can interact with the host system directly.

Use it only if you understand the security implications.

## Quick start

Run the container like this:

```shell
docker run -d \
  --name ttydbridge \
  -e PORT=2222 \
  -v /opt:/opt \
  --pid host \
  --privileged \
  --restart unless-stopped \
  cp0204/ttydbridge:latest
```

Then open the web terminal in your browser:

```text
http://yourhost:2222
```

If `START_COMMAND=login` is used, log in with a valid system user from the host.

## Example: custom terminal colors

You can customize the terminal appearance with `THEME_JSON`.
For example, for a true black background with a bright purple foreground:

```shell
docker run -d \
  --name ttydbridge \
  -e PORT=2222 \
  -e THEME_JSON='{"background":"#000000","foreground":"#c084fc","cursor":"#c084fc","selection":"#4c1d95"}' \
  -v /opt:/opt \
  --pid host \
  --privileged \
  --restart unless-stopped \
  cp0204/ttydbridge:latest
```

## Environment variables

| Name | Default | Description |
|---|---:|---|
| `EXEC_DIR` | `/opt` | Directory used to store the host-side `ttyd` binary and runtime files. Keep this aligned with your volume mapping. |
| `START_COMMAND` | `login` | Initial command passed to `ttyd`. `login` uses host system authentication. You can also use commands such as `bash`. |
| `PORT` | `2222` | Web port used by the browser terminal. The script validates that the value is a valid TCP port. |
| `ALLOW_WRITE` | `true` | Allows interactive keyboard input in the terminal. If set to `false`, the session becomes read-only. |
| `HTTP_USERNAME` / `HTTP_PASSWORD` |  | Enables HTTP basic authentication when both values are set. |
| `ENABLE_SSL` | `false` | Enables HTTPS support in `ttyd`. |
| `SSL_CERT` / `SSL_KEY` / `SSL_CA` |  | Certificate paths on the host. Used only when `ENABLE_SSL=true`. |
| `ENABLE_IPV6` | `false` | Enables IPv6 support for the web terminal. |
| `AUTO_ALLOW_PORT` | `false` | Automatically creates an `iptables` rule on the host to allow inbound access to the selected port. |
| `THEME_JSON` |  | Applies terminal theme colors using ttyd client theme settings. Recommended format: compact JSON. |
| `CUSTOM_OPTIONS` |  | Additional raw ttyd arguments. Useful for passing extra supported options to ttyd. |

## Theme example

The recommended format for `THEME_JSON` is compact JSON:

```shell
THEME_JSON='{"background":"#000000","foreground":"#c084fc","cursor":"#c084fc","selection":"#4c1d95"}'
```

## Custom ttyd options

You can pass additional ttyd arguments with `CUSTOM_OPTIONS`.

Example:

```shell
-e CUSTOM_OPTIONS='--check-origin'
```

Or:

```shell
-e CUSTOM_OPTIONS='--check-origin -t titleFixed=ttydBridge'
```

## Runtime behavior

- On startup, the container copies the `ttyd` binary to the configured host execution directory if needed.
- If `AUTO_ALLOW_PORT=true`, the script adds a matching host firewall rule when necessary.
- A PID file is created at `EXEC_DIR/ttydbridge.pid` so the started host-side `ttyd` process can be stopped more cleanly.
- On shutdown, the script attempts to stop the tracked `ttyd` process and removes temporary resources it created itself.

## Default behavior summary

- **Write access** is enabled by default
- **System login** is used by default
- **SSL** is disabled by default
- **IPv6** is disabled by default
- **Automatic firewall opening** is disabled by default

## Minimal example with authentication

```shell
docker run -d \
  --name ttydbridge \
  -e PORT=2222 \
  -e HTTP_USERNAME=admin \
  -e HTTP_PASSWORD=changeme \
  -v /opt:/opt \
  --pid host \
  --privileged \
  --restart unless-stopped \
  cp0204/ttydbridge:latest
```

## Minimal example with HTTPS

```shell
docker run -d \
  --name ttydbridge \
  -e PORT=2222 \
  -e ENABLE_SSL=true \
  -e SSL_CERT=/path/to/fullchain.pem \
  -e SSL_KEY=/path/to/privkey.pem \
  -v /opt:/opt \
  --pid host \
  --privileged \
  --restart unless-stopped \
  cp0204/ttydbridge:latest
```

## Contribution

I forked this project from Cp0204/ttydBridge to enhance and implement new features beside the original project


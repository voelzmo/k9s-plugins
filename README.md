# k9s-plugins

A collection of useful [k9s](https://k9scli.io) plugins. Each plugin is a
self-contained YAML file in [`plugins/`](plugins/) that you can drop into your
k9s configuration.

## Plugins

| Plugin | Key | Scope | Description |
| ------ | --- | ----- | ----------- |
| [`secret-decode-certs`](plugins/secret-decode-certs.yaml) | `Shift-C` | `secrets` | Decode and display every certificate stored in a Secret |

### secret-decode-certs

Select a Secret in k9s and press `Shift-C`. The plugin walks over all entries
in `.data`, picks those containing a PEM `BEGIN CERTIFICATE` block, and prints
the fully decoded certificate(s) (subject, issuer, validity, SANs, ...) in
`less`. Entries that don't contain certificates are skipped.

**Requires:** `kubectl`, `jq`, `base64`, `less`, and an `openssl` that supports
`storeutl` (OpenSSL 1.1.1 or newer). Note that the `openssl` shipped with macOS
is LibreSSL; if `storeutl` isn't available, install OpenSSL
(e.g. via Homebrew) and make sure it comes first in your `PATH`.

## Installation

k9s loads plugin files from a `plugins` directory next to its config, so no
YAML merging is needed. Where that directory is depends on your platform and
your `XDG_*` settings; run `k9s info` and look at the directory of the
`Plugins:` entry (the `plugins/` directory lives next to `plugins.yaml`).

Typical locations:

- Linux: `~/.config/k9s/plugins/`
- macOS: `~/Library/Application Support/k9s/plugins/` (or
  `$XDG_CONFIG_HOME/k9s/plugins/` if `XDG_CONFIG_HOME` is set)

Then either copy or symlink the plugin(s) you want. Symlinks make it easy to
pick up updates with a `git pull`:

```sh
git clone https://github.com/voelzmo/k9s-plugins.git

# macOS (without XDG_CONFIG_HOME); on Linux use "${XDG_CONFIG_HOME:-$HOME/.config}/k9s/plugins"
K9S_PLUGIN_DIR="$HOME/Library/Application Support/k9s/plugins"

mkdir -p "$K9S_PLUGIN_DIR"
ln -s "$PWD/k9s-plugins/plugins/secret-decode-certs.yaml" "$K9S_PLUGIN_DIR/"
```

Restart k9s afterwards. If you'd rather keep everything in a single
`plugins.yaml`, paste the content of a plugin file under a top-level `plugins:`
key.

## Adding a plugin

1. Create `plugins/<plugin-name>.yaml`. The top-level key is the plugin name
   (no `plugins:` wrapper), for example:

   ```yaml
   my-plugin:
     shortCut: Shift-X
     description: What it does
     scopes:
       - pods
     command: bash
     background: false
     args:
       - -c
       - echo "$NAMESPACE/$NAME"
   ```

2. Add a comment at the top of the file listing its external requirements.
3. Pick a `shortCut` that doesn't clash with k9s built-ins or the other plugins
   listed above.
4. Add a row to the table in this README.

See the [k9s plugin documentation](https://k9scli.io/topics/plugins/) for all
available fields and variables (`$NAMESPACE`, `$NAME`, `$CONTEXT`, ...).

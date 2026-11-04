# k9s-plugins

A collection of useful [k9s](https://k9scli.io) plugins. Each plugin is a
self-contained YAML file in [`plugins/`](plugins/) that you can drop into your
k9s configuration.

## Plugins

| Plugin | Key | Scope | Description |
| ------ | --- | ----- | ----------- |
| [`secret-decode-certs`](plugins/secret-decode-certs.yaml) | `Shift-C` | `secrets` | Show subject, issuer, validity and SANs of every certificate stored in a Secret |

### secret-decode-certs

Highlight a Secret in the `:secrets` list and press `Shift-C`. The plugin walks
over all entries in `.data` and picks those containing a PEM
`BEGIN CERTIFICATE` block. Bundles with several certificates (e.g. a CA chain)
are split, and for each certificate it shows the subject, issuer, validity
dates and Subject Alternative Names in `less`. Entries that don't contain
certificates are skipped.

Example output:

```
=== tls.crt
--- certificate 1
subject=CN = example.com
issuer=CN = Example CA
notBefore=Nov  4 11:37:28 2026 GMT
notAfter=Dec  4 11:37:28 2026 GMT
X509v3 Subject Alternative Name:
    DNS:example.com, DNS:www.example.com, IP Address:10.0.0.1
```

**Requires:** `kubectl`, `jq`, `awk`, `base64`, `less`, and an `openssl` that
supports `x509 -ext` (OpenSSL 1.1.1 or newer). Note that the `openssl` shipped
with macOS is LibreSSL and doesn't support `-ext`; install OpenSSL
(e.g. via Homebrew) and make sure it comes first in your `PATH`.

**Limitation:** k9s only binds plugin shortcuts in list views (and the xray
tree), not inside a resource's YAML, describe or decode view. Use the shortcut
from the list.

## Installation

k9s loads plugin files from a `plugins` directory next to its config, so no
YAML merging is needed (see the
[k9s plugin documentation](https://k9scli.io/topics/plugins/) for details on
how k9s discovers plugins). Where that directory is depends on your platform and
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

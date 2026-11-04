# k9s-plugins

A collection of useful [k9s](https://k9scli.io) plugins. Each plugin is a
self-contained YAML file in [`plugins/`](plugins/) that you can drop into your
k9s configuration.

## Plugins

| Plugin | Key | Scope | Description |
| ------ | --- | ----- | ----------- |
| [`secret-decode-certs`](plugins/secret-decode-certs.yaml) | `Shift-C` | `secrets` | Show subject, issuer, validity and SANs of every certificate stored in a Secret |
| [`secret-decode-brotli`](plugins/secret-decode-brotli.yaml) | `Shift-B` | `secrets` | Decompress and show brotli-compressed entries of a Secret |
| [`blame`](plugins/blame.yaml) | `Shift-M` | `all` | Show who last modified each field of the selected resource (via `managedFields`) |

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

### secret-decode-brotli

Highlight a Secret in the `:secrets` list and press `Shift-B`. The plugin
tries to decompress every entry in `.data` as brotli. Entries that decompress
to something non-empty are shown in `less`, labelled `(brotli)`. If the content
is JSON it is pretty-printed, otherwise it is shown as-is. All other entries are
skipped.

Brotli has no magic bytes, so there is no header to check. Detection works by
attempting decompression, which means that an entry which isn't brotli is
skipped because decompression fails, not because it was recognised. Very short
or unusual values could in theory decompress by accident.

**Requires:** `kubectl`, `jq`, `base64`, `brotli`, `less`.

### blame

Highlight any resource in a list view and press `Shift-M`. The plugin runs
[`kubectl-blame`](https://github.com/knight42/kubectl-blame), which annotates
every line of the resource's YAML with the manager and time of its last change
(from `.metadata.managedFields`), and shows the result in `less`:

```
kubectl-client-side-apply (Update 34d)   a: "1"
argocd-controller         (Update 27h)   b: "2"
```

This is an improved version of the
[upstream k9s plugin](https://github.com/derailed/k9s/blob/master/plugins/blame.yaml):
variables are quoted, `-n` is omitted for cluster-scoped resources, the API
group is passed (`deployments.apps`), and views without a selected resource
(e.g. containers) show a message instead of an error. The shortcut is
`Shift-M` instead of `b`, because plain `b` is already bound by k9s in the
service and port-forward views.

**Requires:** `kubectl-blame` (`kubectl krew install blame`), `less`.

## Shortcuts

Shortcuts used here: `Shift-B` (brotli), `Shift-C` (certs), `Shift-M` (blame).
In k9s v0.51.0 none of them is bound by a built-in view where these plugins
apply. Check before adding a plugin: k9s skips a plugin whose key is already
taken and shows `Plugins load failed!` instead (unless the plugin sets
`override: true`). Plugins from other collections, such as the ones bundled in
the k9s repo, may use the same keys. For example, `argocd.yaml` and
`kubectl-get-in-shell.yaml` both use `Shift-B`.

## Limitations

k9s only binds plugin shortcuts in list views (and the xray tree), not inside a
resource's YAML, describe or decode view. Use the shortcuts from the list.

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

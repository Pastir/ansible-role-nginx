# pastir.nginx

Installs nginx from the official **nginx.org** repository (stable or mainline) with a pinned, held version.
Configuration is the job of [pastir.nginx_config](../pastir.nginx_config).

- The repository is `/etc/apt/sources.list.d/nginx.sources` with the key in `/etc/apt/keyrings`, plus an apt pin
  so the nginx.org origin wins over the Ubuntu archive.
- **Guard:** if nginx is already installed in another version, or is Ubuntu's build (`nginx-common`/`nginx-core`),
  the role stops. Switching requires `-e nginx_allow_upgrade=true`. Ubuntu's packages are then replaced,
  `/etc/nginx` is kept, and nginx restarts.
- **Package `default.conf`:** the nginx.org package ships `/etc/nginx/conf.d/default.conf`. It is loaded before
  your own vhosts and becomes the default server on `:80` (e.g. it wins over a catch-all http→https redirect that
  has no `default_server`) until [pastir.nginx_config](../pastir.nginx_config) removes it
  (`nginx_remove_default_site`). With `nginx_remove_package_default_conf: true` this role removes it right after
  the package install/upgrade, before nginx is started, so the window never opens even if a later role fails.
  The file is removed only if it is the package's untouched copy (its md5 equals the conffile record from
  `dpkg-query -W -f='${Conffiles}' nginx`); a modified file is kept and a warning is printed. If nginx was already
  running (e.g. the package's postinst started it during a takeover), it is reloaded. dpkg does not restore a
  deleted conffile on later upgrades, so the removal sticks; a purge and reinstall brings it back (and the role
  removes it again). In `--check` on a host without nginx.org nginx yet, the removal is not predicted.

## Variables

| Variable | Default | Description |
|---|---|---|
| `nginx_version` | `1.30.6` | Exact version. Empty means the latest of the branch |
| `nginx_branch` | `stable` | `stable` \| `mainline` |
| `nginx_hold` | `true` | `apt-mark hold nginx` |
| `nginx_allow_upgrade` | `false` | Allow a version change or replacing Ubuntu's nginx |
| `nginx_remove_package_default_conf` | `false` | Remove the package's unmodified `conf.d/default.conf` right after install, before nginx starts |
| `nginx_repo_enabled` | `true` | |
| `nginx_repo_url`, `nginx_signing_key_url`, `nginx_keyring` | nginx.org | |
| `nginx_extra_packages` | `[]` | e.g. `nginx-module-geoip` |
| `nginx_sites_dirs` | `true` | Create `sites-available` / `sites-enabled` |
| `nginx_service_enabled` | `true` | |

## Example

```yaml
- hosts: web
  become: true
  roles:
    - role: pastir.nginx
      vars:
        nginx_version: "1.30.6"
        nginx_remove_package_default_conf: true
    - role: pastir.nginx_config
```

## Testing

`molecule test`: clean install on 24.04 and 26.04, a takeover of Ubuntu's nginx 1.24, and the version guard.
`nginx_remove_package_default_conf` is on for 26.04 and the takeover (default.conf gone, nothing on `:80`),
off on 24.04 (file kept); a modified default.conf is kept.

## Installation

```yaml
# requirements.yml
roles:
  - name: pastir.nginx
    src: git+git@github.com:Pastir/ansible-role-nginx.git
    version: main  # or a tag
```

```bash
ansible-galaxy install -r requirements.yml
```

## License

Apache-2.0

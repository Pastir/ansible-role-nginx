# pastir.nginx

Installs nginx from the official **nginx.org** repository (stable or mainline) with a pinned, held version.
Configuration is the job of [pastir.nginx_config](../pastir.nginx_config).

- The repository is `/etc/apt/sources.list.d/nginx.sources` with the key in `/etc/apt/keyrings`, plus an apt pin
  so the nginx.org origin wins over the Ubuntu archive.
- **Guard:** if nginx is already installed in another version, or is Ubuntu's build (`nginx-common`/`nginx-core`),
  the role stops. Switching requires `-e nginx_allow_upgrade=true`. Ubuntu's packages are then replaced,
  `/etc/nginx` is kept, and nginx restarts.

## Variables

| Variable | Default | Description |
|---|---|---|
| `nginx_version` | `1.30.6` | Exact version. Empty means the latest of the branch |
| `nginx_branch` | `stable` | `stable` \| `mainline` |
| `nginx_hold` | `true` | `apt-mark hold nginx` |
| `nginx_allow_upgrade` | `false` | Allow a version change or replacing Ubuntu's nginx |
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
    - role: pastir.nginx_config
```

## Testing

`molecule test`: clean install on 24.04 and 26.04, a takeover of Ubuntu's nginx 1.24, and the version guard.

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

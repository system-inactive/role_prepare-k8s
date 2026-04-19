# prepare-k8s

Ansible role for preparing Debian/Ubuntu nodes for Kubernetes deployment.

Installs and configures containerd, CNI plugins, runc, and Kubernetes packages (kubelet, kubeadm, kubectl). Sets required kernel modules and sysctl parameters.

## What the role does

1. Configures persistent kernel module loading (`overlay`, `br_netfilter`)
2. Applies required sysctl parameters for Kubernetes networking
3. Installs system dependencies
4. Downloads and installs containerd from GitHub releases
5. Downloads and installs CNI plugins
6. Downloads and installs runc
7. Enables and verifies containerd systemd service
8. Adds Kubernetes apt repository and installs kubelet, kubeadm, kubectl

## Requirements

- Debian / Ubuntu (uses `apt`)
- Ansible 2.9+
- Collections:
  - `ansible.posix`

```bash
ansible-galaxy collection install ansible.posix
```

## Role Variables

### Kubernetes

| Variable | Default | Description |
|---|---|---|
| `k8s_version` | `1.31` | Kubernetes version for repo URL |
| `k8s_packages` | `[kubelet, kubeadm, kubectl]` | Packages to install |
| `k8s_repo_key_dest_path` | `/etc/apt/keyrings/kubernetes-apt-keyring.gpg` | Path for apt repo GPG key |
| `k8s_distro_packages_cp` | see defaults | System packages to install |
| `k8s_kernel_modules` | `[overlay, br_netfilter]` | Kernel modules to load persistently |
| `k8s_sysctl_options` | see defaults | sysctl parameters to apply |
| `k8s_sysctl_file` | `/etc/sysctl.d/100-k8s.conf` | Path for sysctl config file |
| `k8s_modload_d_path` | `/etc/modules-load.d/100-k8s.conf` | Path for modules-load config |

### containerd

| Variable | Default | Description |
|---|---|---|
| `containerd_version` | `2.2.0` | containerd version to install |
| `containerd_arch` | `amd64` | Target architecture |
| `containerd_dest_dir` | `/usr/local` | Installation directory |
| `containerd_systemd_file_dest` | `/etc/systemd/system/containerd.service` | Path for systemd unit |
| `containerd_download_checksum` | _(undefined)_ | Optional checksum for archive verification |

### CNI plugins

| Variable | Default | Description |
|---|---|---|
| `containerd_cni_plugins_version` | `1.9.0` | CNI plugins version |
| `containerd_cni_dest_dir` | `/usr/lib/cni` | CNI plugins installation directory |
| `containerd_cni_download_checksum` | _(undefined)_ | Optional checksum for archive verification |

### runc

| Variable | Default | Description |
|---|---|---|
| `runc_version` | `1.4.0` | runc version to install |
| `runc_dest` | `/usr/local/bin/runc` | Installation path |
| `runc_download_checksum` | _(undefined)_ | Optional checksum for binary verification |

## Example Playbook

### Basic usage

```yaml
- hosts: k8s_nodes
  become: true
  roles:
    - role: role_prepare-k8s
```

### With custom versions

```yaml
- hosts: k8s_nodes
  become: true
  roles:
    - role: role_prepare-k8s
      vars:
        k8s_version: "1.31"
        containerd_version: "2.2.0"
        runc_version: "1.4.0"
        containerd_cni_plugins_version: "1.9.0"
```

### With checksum verification

```yaml
- hosts: k8s_nodes
  become: true
  roles:
    - role: role_prepare-k8s
      vars:
        containerd_download_checksum: "sha256:abc123..."
        containerd_cni_download_checksum: "sha256:def456..."
        runc_download_checksum: "sha256:ghi789..."
```

## Tags

Role tasks support selective execution via tags:

| Tag | Tasks |
|---|---|
| `dependencies` | Kernel modules, sysctl, system packages |
| `containerd` | containerd, CNI plugins, runc |
| `k8s-install` | Kubernetes repo and packages |

```bash
# Install only containerd
ansible-playbook site.yml --tags containerd

# Skip k8s packages (prepare node only)
ansible-playbook site.yml --skip-tags k8s-install
```

## Notes

- Temporary download files are placed in `~/.ansible/downloads` on the target host and are not removed automatically after installation
- containerd and CNI plugins are installed with `creates:` guard — re-running the role will not reinstall if binaries already exist
- The role targets **Debian/Ubuntu** only (uses `apt` and `apt_repository`)
- `keepalived` and `haproxy` are included in `k8s_distro_packages_cp` for HA control-plane setups — remove them from the list if not needed

## License

MIT

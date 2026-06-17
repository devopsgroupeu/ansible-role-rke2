# Configuration Reference

All variables are defined in `defaults/main.yml` and can be overridden in your playbook,
`group_vars`, or `host_vars`. Every variable uses the `rke2_` prefix.

---

## RKE2 Version

| Variable | Default | Description |
|---|---|---|
| `rke2_version` | `v1.28.15+rke2r1` | RKE2 version to install. See [GitHub releases](https://github.com/rancher/rke2/releases). |

---

## Installation Mode

| Variable | Default | Description |
|---|---|---|
| `rke2_airgapped` | `false` | Set `true` for air-gapped environments. |
| `rke2_airgapped_artifacts_dir` | `""` | Path on the Ansible controller to pre-downloaded RKE2 artifacts. See [Air-Gapped Install](../README.md#air-gapped-install). |

**Required artifacts** (place in `rke2_airgapped_artifacts_dir`):

```
rke2-install.sh
rke2.linux-amd64.tar.gz
sha256sum-amd64.txt
rke2-images.linux-amd64.tar.zst   # optional — for fully offline image loading
```

---

## Proxy

| Variable | Default | Description |
|---|---|---|
| `rke2_http_proxy` | `""` | HTTP proxy URL for RKE2 services. |
| `rke2_https_proxy` | `""` | HTTPS proxy URL for RKE2 services. |
| `rke2_no_proxy` | `""` | Comma-separated list of hosts/CIDRs that bypass the proxy. |
| `rke2_containerd_http_proxy` | `""` | HTTP proxy for containerd (image pulls). |
| `rke2_containerd_https_proxy` | `""` | HTTPS proxy for containerd. |
| `rke2_containerd_no_proxy` | `""` | No-proxy list for containerd. |

**Example:**

```yaml
rke2_http_proxy: "http://proxy.example.com:3128"
rke2_https_proxy: "http://proxy.example.com:3128"
rke2_no_proxy: "localhost,127.0.0.1,10.0.0.0/8,192.168.0.0/16"
```

---

## CNI Plugin

| Variable | Default | Description |
|---|---|---|
| `rke2_cni` | `"default"` | CNI plugin: `canal`, `cilium`, `calico`, `flannel`, or `default` (canal). |
| `rke2_cni_configs` | `{}` | CNI-specific HelmChartConfig values. Keyed by CNI name. |

**Example — Calico with jumbo frames:**

```yaml
rke2_cni: calico
rke2_cni_configs:
  calico:
    installation:
      calicoNetwork:
        mtu: 9000
```

---

## VIP / High Availability

| Variable | Default | Description |
|---|---|---|
| `rke2_vip_enabled` | `false` | Enable VIP for a stable API endpoint across control plane nodes. |
| `rke2_vip_address` | `""` | The virtual IP address (e.g. `10.0.0.100`). Added to `tls-san` automatically. |
| `rke2_vip_interface` | `"eth0"` | Network interface for the VIP on control plane nodes. |
| `rke2_vip_manager` | `"kubevip"` | VIP manager: `kubevip` or `keepalived`. |
| `rke2_kubevip_version` | `"v0.8.0"` | kube-vip image tag (only when `rke2_vip_manager=kubevip`). |
| `rke2_keepalived_router_id` | `51` | VRRP virtual router ID — must be unique per subnet (1-255). |
| `rke2_keepalived_priority_master` | `101` | VRRP priority for the first server node (MASTER). |
| `rke2_keepalived_priority_backup` | `100` | VRRP priority for additional server nodes (BACKUP). |
| `rke2_keepalived_auth_pass` | `"rke2vip"` | VRRP authentication password (max 8 characters). |

**kube-vip example:**

```yaml
rke2_vip_enabled: true
rke2_vip_address: "10.0.0.100"
rke2_vip_interface: "ens3"
rke2_vip_manager: kubevip
rke2_kubevip_version: "v0.8.0"
```

**keepalived example:**

```yaml
rke2_vip_enabled: true
rke2_vip_address: "10.0.0.100"
rke2_vip_interface: "ens3"
rke2_vip_manager: keepalived
rke2_keepalived_router_id: 51
rke2_keepalived_auth_pass: "s3cur3pw"
```

---

## Cluster Networking

| Variable | Default | Description |
|---|---|---|
| `rke2_tls_san` | `[]` | Additional TLS SANs for the API server (hostnames or IPs). The VIP is added automatically. |
| `rke2_cluster_cidr` | `""` | Pod network CIDR. RKE2 default: `10.42.0.0/16`. |
| `rke2_service_cidr` | `""` | Service network CIDR. RKE2 default: `10.43.0.0/16`. |

---

## etcd Snapshots

| Variable | Default | Description |
|---|---|---|
| `rke2_etcd_snapshot` | `false` | Enable scheduled etcd snapshots. |
| `rke2_etcd_snapshot_schedule` | `""` | Cron expression for snapshot schedule (e.g. `"0 */6 * * *"`). |

---

## Data and Kubeconfig

| Variable | Default | Description |
|---|---|---|
| `rke2_data_dir` | `"/var/lib/rancher/rke2"` | RKE2 data directory. |
| `rke2_write_kubeconfig_mode` | `"0600"` | File mode for the generated kubeconfig. |
| `rke2_kubeconfig_download` | `false` | Download kubeconfig to the Ansible controller after install. |
| `rke2_kubeconfig_output_path` | `"{{ playbook_dir }}/rke2.yaml"` | Local path on the controller where kubeconfig is saved. |

---

## Node Configuration

| Variable | Default | Description |
|---|---|---|
| `rke2_server_nodes` | `[]` | Per-server-node labels and taints (matched by `ansible_host`). |
| `rke2_agent_nodes` | `[]` | Per-agent-node labels and taints (matched by `ansible_host`). |

**Example:**

```yaml
rke2_server_nodes:
  - host: 10.0.0.1
    node-taint:
      - "CriticalAddonsOnly=true:NoExecute"

rke2_agent_nodes:
  - host: 10.0.0.10
    node-label:
      - "role=worker"
      - "zone=a"
```

---

## Custom CA Certificates

| Variable | Default | Description |
|---|---|---|
| `rke2_custom_root_ca_cert` | `""` | Base64-encoded root CA certificate. Store with Ansible Vault. |
| `rke2_custom_root_ca_key` | `""` | Base64-encoded root CA private key. |
| `rke2_custom_intermediate_ca_cert` | `""` | Base64-encoded intermediate CA certificate (optional). |
| `rke2_custom_intermediate_ca_key` | `""` | Base64-encoded intermediate CA private key (optional). |

---

## Certificate Rotation

| Variable | Default | Description |
|---|---|---|
| `rke2_cert_rotation_enabled` | `false` | Enable automated certificate rotation via cron. |
| `rke2_cert_rotation_expiry_threshold_days` | `20` | Rotate when a certificate expires within N days. |
| `rke2_cert_rotation_schedule_type` | `"weekly"` | Schedule type: `daily`, `weekly`, or `monthly`. |
| `rke2_cert_rotation_schedule_hour` | `"20"` | Hour to run rotation (0-23). |
| `rke2_cert_rotation_schedule_minute` | `"0"` | Minute to run rotation (0-59). |
| `rke2_cert_rotation_schedule_day_of_week` | `"1"` | Day of week (1=Monday) — used when `type=weekly`. |
| `rke2_cert_rotation_schedule_day_of_month` | `"1"` | Day of month — used when `type=monthly`. |
| `rke2_cert_rotation_log_retention_days` | `7` | How many days to keep rotation logs. |

---

## Helm Addons

| Variable | Default | Description |
|---|---|---|
| `rke2_addons` | `[]` | List of Helm addons to deploy via RKE2's built-in HelmChart CRD. |

Each addon entry supports:

| Key | Required | Description |
|---|---|---|
| `name` | Yes | Addon identifier (used as manifest filename). |
| `enabled` | Yes | `true` to deploy, `false` to remove. |
| `repo` | Yes | Helm chart repository URL. |
| `chart` | No | Chart name (defaults to `name`). |
| `version` | Yes | Chart version to deploy. |
| `namespace` | Yes | Target Kubernetes namespace. |
| `values` | No | Dict of Helm values to pass to the chart. |

**Example:**

```yaml
rke2_addons:
  - name: cert-manager
    enabled: true
    repo: "https://charts.jetstack.io"
    chart: cert-manager
    version: "v1.16.2"
    namespace: cert-manager
    values:
      crds:
        enabled: true
```

---

## Private Registry

| Variable | Default | Description |
|---|---|---|
| `rke2_private_registries_config` | `""` | YAML string for `/etc/rancher/rke2/registries.yaml`. |

**Example:**

```yaml
rke2_private_registries_config: |
  mirrors:
    docker.io:
      endpoint:
        - "https://registry.example.com:5000"
  configs:
    "registry.example.com:5000":
      auth:
        username: myuser
        password: mypassword
      tls:
        insecure_skip_verify: false
```

---

## Additional Packages

| Variable | Default | Description |
|---|---|---|
| `rke2_additional_packages` | `[curl, iptables]` | System packages installed before RKE2. `curl` and `iptables` are required. |

---

## Cluster Token

| Variable | Default | Description |
|---|---|---|
| `rke2_cluster_token` | `""` | Pre-set cluster token. When empty the role reads the auto-generated `node-token` from the first server. Set this to a value from Vault or a secrets manager for fully declarative, auditable cluster tokens. |
| `rke2_allow_downgrade` | `false` | Allow installing a version older than what is currently installed. Default `false` protects against accidental rollbacks — the role fails with a clear message if a downgrade is detected. |

---

## Built-in Component Management

Disable RKE2 built-in components when you want to replace them with your own (e.g. use a custom ingress controller instead of `rke2-ingress-nginx`).

| Variable | Default | Description |
|---|---|---|
| `rke2_disable` | `[]` | List of built-in components to disable. |

Available components: `rke2-ingress-nginx`, `rke2-metrics-server`, `rke2-coredns`, `rke2-snapshot-controller`, `rke2-snapshot-validation-webhook`

```yaml
rke2_disable:
  - rke2-ingress-nginx
  - rke2-metrics-server
```

---

## Kubernetes Component Arguments

Pass extra flags to individual Kubernetes components without modifying role internals.
Each list entry is a string in `"key=value"` format.

| Variable | Default | Description |
|---|---|---|
| `rke2_kube_apiserver_args` | `[]` | Extra flags for `kube-apiserver`. |
| `rke2_kube_scheduler_args` | `[]` | Extra flags for `kube-scheduler`. |
| `rke2_kube_controller_manager_args` | `[]` | Extra flags for `kube-controller-manager`. |
| `rke2_kubelet_args` | `[]` | Extra flags for `kubelet` (applies to both server and agent nodes). |
| `rke2_kube_proxy_args` | `[]` | Extra flags for `kube-proxy` (applies to both server and agent nodes). |

```yaml
rke2_kube_apiserver_args:
  - "audit-log-maxage=30"
  - "audit-log-maxbackup=10"

rke2_kubelet_args:
  - "max-pods=200"
  - "eviction-hard=memory.available<200Mi"
```

---

## Hardening

| Variable | Default | Description |
|---|---|---|
| `rke2_cis_profile` | `""` | CIS Kubernetes benchmark profile. Options: `cis` (RKE2 ≥ 1.25) or `cis-1.23` (legacy). Activates RKE2's built-in hardened defaults for all control plane components. |
| `rke2_selinux` | `false` | Enable SELinux mode in RKE2 and containerd. On RHEL-family nodes the role automatically installs `container-selinux`. When `false` on RHEL, SELinux is set to `permissive`. |

```yaml
rke2_cis_profile: "cis"
rke2_selinux: true
```

---

## Runtime Options

| Variable | Default | Description |
|---|---|---|
| `rke2_node_name` | `""` | Override the Kubernetes node name. Defaults to the system hostname when empty. Useful when hostnames are not unique or not DNS-resolvable. |
| `rke2_debug` | `false` | Enable RKE2 debug logging. Very verbose — use only for troubleshooting. |

---

## Extra Configuration (catch-all)

`rke2_extra_config` accepts any key/value pairs that are merged into `config.yaml` **after** all other settings. Use this for RKE2 options not yet exposed as dedicated role variables. Values here take precedence over everything above.

```yaml
rke2_extra_config:
  protect-kernel-defaults: true
  snapshotter: "overlayfs"
  container-runtime-endpoint: "unix:///run/containerd/containerd.sock"
```

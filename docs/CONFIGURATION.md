# Configuration Reference

All variables are defined in `defaults/main.yml` and can be overridden in your playbook,
`group_vars`, or `host_vars`. Every variable uses the `rke2_` prefix.

> Full type and constraint information (choices, element types, defaults) is in
> `meta/argument_specs.yml`. This file covers the most-used variables; the complete
> list is in `defaults/main.yml`.

---

## RKE2 Version

| Variable | Default | Description |
|---|---|---|
| `rke2_version` | `v1.36.1+rke2r2` | RKE2 version to install. See [GitHub releases](https://github.com/rancher/rke2/releases). Takes precedence over `rke2_channel`. |
| `rke2_channel` | `""` | Install channel used only when `rke2_version` is empty. Options: `stable`, `latest`, or a per-minor channel like `"v1.35"`. |

---

## Installation Mode

| Variable | Default | Description |
|---|---|---|
| `rke2_airgapped` | `false` | Set `true` for air-gapped environments. |
| `rke2_airgapped_artifacts_dir` | `""` | Path on the Ansible controller to pre-downloaded RKE2 artifacts. |
| `rke2_require_artifacts_dir` | `true` | Require `rke2_airgapped_artifacts_dir` when `rke2_airgapped` is true. Set `false` for a deliberate partial air-gap. |
| `rke2_disable_firewalld` | `true` | Stop and disable firewalld before install. Set `false` to manage firewall rules yourself. |
| `rke2_system_default_registry` | `""` | Mirror registry for all RKE2 system images (e.g. `registry.example.com:5000`). Honored on servers and agents. |

**Required artifacts** (place in `rke2_airgapped_artifacts_dir`):

```
rke2-install.sh
rke2.linux-amd64.tar.gz
sha256sum-amd64.txt
rke2-images.linux-amd64.tar.zst   # optional — for fully offline image loading
```

---

## Cluster Token

| Variable | Default | Description |
|---|---|---|
| `rke2_cluster_token` | `""` | Pre-set cluster token. When empty the role reads the auto-generated `node-token` from the first server. Set this to a value from Vault for a fully declarative, auditable cluster token. |
| `rke2_agent_token` | `""` | Optional separate token for agent nodes. When set, agents use this instead of `rke2_cluster_token` (less cluster access). |
| `rke2_allow_downgrade` | `false` | Allow installing a version older than what is currently installed. Default `false` protects against accidental rollbacks. |

---

## Proxy

| Variable | Default | Description |
|---|---|---|
| `rke2_http_proxy` | `""` | HTTP proxy URL for RKE2 services. |
| `rke2_https_proxy` | `""` | HTTPS proxy URL for RKE2 services. |
| `rke2_no_proxy` | `""` | Comma-separated list of hosts/CIDRs that bypass the proxy. |
| `rke2_containerd_http_proxy` | `""` | HTTP proxy for containerd (image pulls). Overrides `rke2_http_proxy` for containerd only. |
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
| `rke2_agent_server_address` | `""` | Override the server address in the agent `config.yaml` `server` field. Defaults to `rke2_vip_address` (when VIP enabled) or the first server's `ansible_host`. Useful on cloud SDN (e.g. Hetzner private networks) where gratuitous-ARP VIPs are not directly routable. |

### kube-vip tuning (when `rke2_vip_manager=kubevip`)

| Variable | Default | Description |
|---|---|---|
| `rke2_kubevip_version` | `"v1.2.0"` | kube-vip image tag. |
| `rke2_kubevip_port` | `"6443"` | Control-plane port kube-vip advertises the VIP on. |
| `rke2_kubevip_svc_enable` | `false` | Enable kube-vip watching of LoadBalancer-type Services (`svc_enable`). Automatically set when `rke2_kubevip_cloud_provider_enabled` is `true`. |
| `rke2_kubevip_service_election_enable` | `false` | Enable per-service leader election (`svc_election`). |
| `rke2_kubevip_metrics_port` | `0` | Port for kube-vip Prometheus metrics. `0` disables the metrics endpoint. |
| `rke2_kubevip_cloud_provider_enabled` | `false` | Deploy the kube-vip cloud provider for LoadBalancer-type Services. Requires `rke2_kubevip_load_balancer_ip_range`. |
| `rke2_kubevip_cloud_provider_image` | `ghcr.io/kube-vip/kube-vip-cloud-provider:v0.0.12` | Image for the kube-vip cloud provider StatefulSet. |
| `rke2_kubevip_load_balancer_ip_range` | `""` | IP pool for LoadBalancer Services. Accepts CIDR (`192.168.0.200/29`) or range (`192.168.0.200-192.168.0.250`). |

**kube-vip example:**

```yaml
rke2_vip_enabled: true
rke2_vip_address: "10.0.0.100"
rke2_vip_interface: "ens3"
rke2_vip_manager: kubevip
rke2_kubevip_version: "v1.2.0"
```

### keepalived VRRP (when `rke2_vip_manager=keepalived`)

| Variable | Default | Description |
|---|---|---|
| `rke2_keepalived_router_id` | `51` | VRRP virtual router ID — must be unique per VRRP group on the subnet (1-255). |
| `rke2_keepalived_priority_master` | `101` | VRRP priority for the first server node (MASTER state). |
| `rke2_keepalived_priority_backup` | `100` | VRRP priority for additional server nodes (BACKUP state). |
| `rke2_keepalived_auth_pass` | `"rke2vip"` | VRRP authentication password (max 8 characters). Store with Ansible Vault. |

**keepalived example:**

```yaml
rke2_vip_enabled: true
rke2_vip_address: "10.0.0.100"
rke2_vip_interface: "ens3"
rke2_vip_manager: keepalived
rke2_keepalived_router_id: 51
rke2_keepalived_auth_pass: "s3cur3pw"
```

### Keepalived floating IP failover

| Variable | Default | Description |
|---|---|---|
| `rke2_keepalived_failover_enabled` | `false` | Enable cloud floating IP failover via keepalived `notify_master` hook. |
| `rke2_keepalived_failover_script` | `"/etc/keepalived/notify-master.sh"` | Absolute path on target nodes where the notify script is deployed (mode 0750). |
| `rke2_keepalived_failover_script_content` | `""` | Shell script body executed when this node becomes MASTER. Sources `failover.env` from the same directory. |
| `rke2_keepalived_failover_env` | `{}` | Key/value pairs written to a protected env file (mode 0600). Use for API tokens and IDs that must not be embedded in the script body. |

---

## Cluster Networking

| Variable | Default | Description |
|---|---|---|
| `rke2_tls_san` | `[]` | Additional TLS SANs for the API server (hostnames or IPs). The VIP is added automatically when `rke2_vip_enabled` is true. |
| `rke2_cluster_cidr` | `""` | Pod network CIDR. RKE2 default: `10.42.0.0/16`. |
| `rke2_service_cidr` | `""` | Service network CIDR. RKE2 default: `10.43.0.0/16`. |
| `rke2_cluster_domain` | `""` | Cluster domain (e.g. `cluster.example.net`). Defaults to `cluster.local` when empty. |
| `rke2_cluster_dns` | `""` | Override the cluster DNS service IP. Must fall inside `rke2_service_cidr`. Empty = RKE2 default. |
| `rke2_node_ip` | `""` | Global node-ip advertised to Kubernetes. Per-node value in `rke2_server_nodes`/`rke2_agent_nodes` overrides this. Empty = auto-detect. |
| `rke2_node_external_ip` | `""` | Global node-external-ip. Per-node value overrides this. Empty = not set. |
| `rke2_disable_kube_proxy` | `false` | Disable kube-proxy entirely (e.g. Cilium in kube-proxy replacement mode). |

---

## etcd Snapshots

| Variable | Default | Description |
|---|---|---|
| `rke2_etcd_disable_snapshots` | `false` | Disable RKE2's built-in etcd snapshots. Snapshots are ON by default (every 12h); set `true` to turn them off entirely. |
| `rke2_etcd_snapshot_schedule` | `""` | Cron expression overriding the snapshot schedule (e.g. `"0 */6 * * *"`). Ignored when `rke2_etcd_disable_snapshots` is true. |
| `rke2_etcd_snapshot_retention` | `5` | Number of local etcd snapshots to retain. |

### S3-compatible snapshot upload

| Variable | Default | Description |
|---|---|---|
| `rke2_etcd_s3_enabled` | `false` | Enable off-cluster S3-compatible etcd snapshot upload. |
| `rke2_etcd_s3_endpoint` | `""` | S3 endpoint host (e.g. `s3.amazonaws.com` or `minio.example.com:9000`). |
| `rke2_etcd_s3_bucket` | `""` | S3 bucket name for etcd snapshots. |
| `rke2_etcd_s3_region` | `""` | S3 region. Defaults to `us-east-1` in RKE2 when empty. |
| `rke2_etcd_s3_folder` | `""` | Optional path prefix inside the S3 bucket. |
| `rke2_etcd_s3_access_key` | `""` | S3 access key. Source from Vault; rendered with `no_log`. |
| `rke2_etcd_s3_secret_key` | `""` | S3 secret key. Source from Vault; rendered with `no_log`. |
| `rke2_etcd_s3_endpoint_ca` | `""` | Optional PEM CA bundle for a private S3 endpoint. |
| `rke2_etcd_s3_skip_ssl_verify` | `false` | Skip TLS verification of the S3 endpoint (lab/self-signed only). |

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

Variables expect **Base64-encoded PEM** input. Pipe raw PEM through `| b64encode`:

```yaml
rke2_custom_root_ca_cert: "{{ lookup('file', 'root-ca.pem') | b64encode }}"
rke2_custom_root_ca_key: "{{ lookup('file', 'root-ca.key') | b64encode }}"
```

| Variable | Default | Description |
|---|---|---|
| `rke2_custom_root_ca_cert` | `""` | Base64-encoded PEM root CA certificate. When set with `rke2_custom_root_ca_key`, the role generates RKE2 cluster certificates signed by this CA on the first server node. Store with Ansible Vault. |
| `rke2_custom_root_ca_key` | `""` | Base64-encoded PEM private key for the root CA. |
| `rke2_custom_intermediate_ca_cert` | `""` | Base64-encoded PEM intermediate CA certificate (optional). |
| `rke2_custom_intermediate_ca_key` | `""` | Base64-encoded PEM private key for the intermediate CA (optional). |
| `rke2_custom_ca_remove_signing_keys` | `true` | Remove root-ca.key and intermediate-ca.key from nodes after the cluster CA chain is generated. Re-supply from Vault for a later CA rotation. |

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

## Ingress Controller

| Variable | Default | Description |
|---|---|---|
| `rke2_ingress_controller` | `"traefik"` | Ingress controller this role configures: `traefik` (default in RKE2 v1.36+), `ingress-nginx` (legacy, EOL Mar 2026), or `none` (disables both built-ins). |
| `rke2_traefik_config` | `""` | YAML string of Helm values for the bundled `rke2-traefik` chart, applied as a `HelmChartConfig` when `rke2_ingress_controller=traefik`. |
| `rke2_ingress_nginx_config` | `""` | YAML string of Helm values for `rke2-ingress-nginx`. Applied when `rke2_ingress_controller=ingress-nginx`. |

---

## Helm Addons

| Variable | Default | Description |
|---|---|---|
| `rke2_addons` | `[]` | List of Helm addons to deploy via RKE2's built-in HelmChart CRD. |
| `rke2_addons_delete_on_disable` | `false` | When `true`, disabling an addon (`enabled: false`) also runs `kubectl delete helmchart <name> -n kube-system`, uninstalling the running release. Default `false` only removes the manifest file. |
| `rke2_helmchartconfigs` | `[]` | `HelmChartConfig` overrides for RKE2 bundled charts. Each entry needs `name` (bundled chart name) and `valuesContent` (YAML string of Helm values). |

Each `rke2_addons` entry supports:

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

rke2_helmchartconfigs:
  - name: rke2-coredns
    valuesContent: |
      replicas: 2
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

## Built-in Component Management

Disable RKE2 built-in components when you want to replace them (e.g. use a custom ingress controller).

| Variable | Default | Description |
|---|---|---|
| `rke2_disable` | `[]` | List of built-in components to disable. |
| `rke2_disable_kube_proxy` | `false` | Disable kube-proxy entirely (for Cilium kube-proxy replacement mode). |

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

## Cloud Provider

| Variable | Default | Description |
|---|---|---|
| `rke2_disable_cloud_controller` | `false` | Disable RKE2's built-in cloud controller manager. Required when using an external cloud controller (e.g. kube-vip cloud provider, AWS CCM). |
| `rke2_cloud_provider_name` | `""` | Cloud provider name passed to kubelet and the API server. Set to `"external"` when using any external cloud controller manager. |

---

## Rolling Upgrades

| Variable | Default | Description |
|---|---|---|
| `rke2_drain_node_during_upgrade` | `false` | Cordon and drain each server node before restarting during upgrades. Requires the play to use `serial: 1`. |
| `rke2_drain_additional_args` | `"--timeout=120s"` | Additional arguments passed to `kubectl drain`. |
| `rke2_drain_force` | `false` | Pass `--force` to `kubectl drain`, evicting pods not managed by a ReplicaSet/Job/DaemonSet. Off by default. |
| `rke2_wait_for_all_pods_to_be_ready` | `false` | After uncordoning a node, wait for all pods to reach Running or Succeeded phase before proceeding to the next node. |

---

## Hardening

| Variable | Default | Description |
|---|---|---|
| `rke2_cis_profile` | `""` | CIS Kubernetes benchmark profile. Options: `cis` (RKE2 ≥ 1.25) or `cis-1.23` (legacy). Activates RKE2's built-in hardened defaults for all control plane components. |
| `rke2_selinux` | `false` | Enable SELinux mode in RKE2 and containerd. On RHEL-family nodes the role automatically installs `container-selinux`. When `false` on RHEL, SELinux is set to `permissive`. |
| `rke2_audit_policy` | `""` | Full YAML body of a Kubernetes audit policy. When set, the role deploys `/etc/rancher/rke2/audit-policy.yaml` (mode 0600) and emits `audit-policy-file` in `config.yaml`. |

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

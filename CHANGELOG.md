# Changelog

All notable changes to this project are documented in this file.

## [1.5.0] - 2026-02-24

### Added
- `rke2_keepalived_failover_enabled`, `rke2_keepalived_failover_script`, `rke2_keepalived_failover_script_content`, `rke2_keepalived_failover_env` — provider-agnostic cloud floating IP failover hook via keepalived `notify_master`; deploys a protected env file (mode 0600) and a notify script on each server node
- README: Cloud Floating IP Failover section with full Hetzner Cloud example
- `meta/argument_specs.yml`: added specs for all 4 failover variables

### Fixed
- `tasks/vip.yml`: health check script task rewritten to use explicit `src`/`dest` pairs instead of `regex_replace` on the loop variable — eliminates ambiguous backslash escaping that caused scripts to be deployed with the `.j2` extension, disabling all keepalived health checks silently

## [1.4.0] - 2026-02-23

### Added
- `meta/argument_specs.yml` — documents all role variables for Galaxy autodoc and `ansible-lint` production profile
- `templates/kube-vip-rbac.yml.j2` — ServiceAccount, ClusterRole, ClusterRoleBinding for kube-vip DaemonSet
- `templates/kube-vip-cloud-controller.yml.j2` — kube-vip cloud provider StatefulSet + RBAC for LoadBalancer-type Services
- `templates/kube-vip-cloud-configmap.yml.j2` — IP pool ConfigMap; auto-detects CIDR vs range notation
- `templates/check-apiserver.sh.j2` + `check-rke2server.sh.j2` — keepalived health check scripts that curl `/healthz` and port 9345
- `tasks/rolling_restart.yml` — cordon → drain → restart → wait → uncordon per node; use with `serial: 1`
- `rke2_kubevip_cloud_provider_enabled`, `rke2_kubevip_cloud_provider_image`, `rke2_kubevip_load_balancer_ip_range` — opt-in kube-vip cloud provider
- `rke2_kubevip_svc_enable`, `rke2_kubevip_service_election_enable`, `rke2_kubevip_metrics_port` — kube-vip DaemonSet tuning variables
- `rke2_agent_server_address` — override server address in agent config (workaround for Hetzner SDN where gratuitous-ARP VIPs are not routable)
- `rke2_agent_token` — separate less-privileged token for agent nodes
- `rke2_disable_cloud_controller`, `rke2_cloud_provider_name` — support for external cloud controller managers
- `rke2_cluster_domain` — configurable cluster domain
- `rke2_disable_kube_proxy` — disable kube-proxy for CNI kube-proxy replacement mode (e.g. Cilium)
- `rke2_drain_node_during_upgrade`, `rke2_drain_additional_args`, `rke2_wait_for_all_pods_to_be_ready` — rolling upgrade variables
- GitHub community files: ISSUE_TEMPLATE (bug + feature), PULL_REQUEST_TEMPLATE, dependabot
- GitHub Actions split into focused workflows: `lint.yml`, `molecule.yml`, `galaxy.yml`, `release-drafter.yml`, `pre-commit.yml`

### Changed
- **kube-vip now deployed as a DaemonSet** via `server/manifests/` instead of a static pod — static pods in RKE2 do not receive a ServiceAccount token, causing CrashLoopBackOff with kube-vip ≥ v0.7.0
- `tasks/vip.yml` — deploys kube-vip RBAC + DaemonSet + optional cloud provider manifests; removes legacy static pod on upgrade; deploys keepalived health check scripts
- `templates/keepalived.conf.j2` — now tracks `chk_apiserver` and `chk_rke2server` scripts instead of bare `systemctl is-active`
- `tasks/proxy_setup.yml` — removed `debug + notify + changed_when: true` anti-pattern; `notify: Reload systemd` moved to template/file tasks directly
- `handlers/main.yml` — migrated from deprecated `ansible.builtin.systemd` to `ansible.builtin.systemd_service`
- `molecule/ha/verify.yml` — updated kube-vip assertions to check DaemonSet in `server/manifests/` (not legacy static pod path); added cloud provider, RBAC, agent token, svc_enable checks
- `README.md` — complete rewrite covering all features, VIP architecture diagrams, cloud provider, rolling upgrades, agent token

### Fixed
- kube-vip CrashLoopBackOff on RKE2 (static pod had no ServiceAccount token)
- kube-vip cloud provider non-functional when `svc_enable` was not set on the DaemonSet
- `ansible-lint` production profile warnings on handlers and meta schema

## [1.3.0] - 2026-01-15

### Added
- Certificate rotation support (`tasks/certs_rotation.yml`, configurable via cron)
- Custom CA certificate injection (`tasks/custom_ca_certs.yml`)
- Helm Addons support (`tasks/addons.yml`) — deploy ArgoCD, cert-manager, or any Helm chart via RKE2 HelmChart CRD
- keepalived support for VIP management (`rke2_vip_manager: keepalived`)
- Air-gapped installation support (`rke2_airgapped`, `rke2_airgapped_artifacts_dir`)
- Private registry configuration (`rke2_private_registries_config`)
- Downgrade protection (`rke2_allow_downgrade`)
- Molecule HA scenario

## [1.2.0] - 2025-12-10

### Fixed
- HA bootstrap: wait for node-token before slurp (was failing on slow systems)
- HA bootstrap: wait for port 9345 on first server before additional nodes join
- kube-vip: corrected static pod path to `agent/pod-manifests/` on all server nodes
- Agent idempotency: version check now happens before install script download

## [1.1.0] - 2025-11-20

### Added
- HA support: multiple server nodes with token distribution
- kube-vip static pod deployment on control-plane nodes
- Kubeconfig download to Ansible controller (`rke2_kubeconfig_download`)
- Proxy configuration (`rke2_http_proxy`, `rke2_https_proxy`, `rke2_no_proxy`)
- SELinux support for RHEL-family

## [1.0.0] - 2025-10-01

### Added
- Initial release
- Single-node RKE2 server installation
- RKE2 agent installation
- Internet install via `get.rke2.io`
- Configurable RKE2 version, CNI, TLS SANs, data directory

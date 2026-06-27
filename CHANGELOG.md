# Changelog

All notable changes to this project are documented in this file.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2026-06-27

### Fixed
- Custom CA injection now runs when a root **certificate** is supplied together
  with an intermediate CA (cert + key), not only when the root **private key** is
  present. This supports keeping the root key offline (e.g. in HashiCorp Vault)
  and seeding RKE2 with a Vault-issued intermediate CA. `generate_ca.sh` already
  signs leaf CAs with the intermediate; only the `when:` gate was too strict.
- The root CA private key file is no longer written as an empty file when
  `rke2_custom_root_ca_key` is unset.

## [1.0.0] - 2026-06-23

Initial public release on the devopsgroupeu Ansible Galaxy namespace. Installs and
configures RKE2 (single-node or HA) with kube-vip or keepalived VIP management,
Helm addons, custom CA injection, certificate rotation, air-gapped installation,
private registries, and CIS hardening.

### Notes for users of the legacy devopsgroup.* roles
- Namespace changed from `devopsgroup` to `devopsgroupeu` — update your
  `requirements.yml` role source and any `meta/main.yml` dependency references.
- `min_ansible_version` is `2.19`; ansible-core < 2.19 is not supported.
- Custom CA variables (`rke2_custom_root_ca_cert`, `rke2_custom_root_ca_key`,
  `rke2_custom_intermediate_ca_cert`, `rke2_custom_intermediate_ca_key`) take
  **Base64-encoded** PEM input — pipe raw PEM through `| b64encode`.
- `rke2_cis_profile` accepts `""`, `cis`, or `cis-1.23`.

> Pre-Galaxy internal development history (versions 1.0.0–1.5.0 under the legacy
> `devopsgroup` namespace) is preserved in the git history.

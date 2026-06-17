# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| 1.x (latest) | Yes |
| < 1.0 | No |

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues.**

If you discover a security vulnerability in this role, please report it by emailing:

**security@devopsgroup.sk**

Include the following in your report:

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if known)

You will receive a response within **5 business days** acknowledging receipt of your report.
We aim to release a fix within **30 days** of confirmation.

## Security Considerations

When using this role, please be aware of the following:

- **`rke2_keepalived_auth_pass`** — store this value with Ansible Vault, not in plaintext.
- **`rke2_custom_root_ca_key` / `rke2_custom_intermediate_ca_key`** — these are private keys; always encrypt with Ansible Vault.
- **`rke2_private_registries_config`** — may contain registry credentials; encrypt with Ansible Vault.
- The kubeconfig file (`rke2.yaml`) provides cluster-admin access — protect it accordingly.
- RKE2 node token (`/var/lib/rancher/rke2/server/node-token`) grants cluster join access — restrict file permissions.

## Install trust model

This role obtains the RKE2 installer and binaries via two paths:

- **Internet install:** the installer is fetched over **TLS** from
  `https://get.rke2.io`; the installer itself downloads the RKE2 tarball and
  verifies it against the upstream `sha256sum-<arch>.txt`. The role does not add
  GPG/cosign signature verification — TLS + the upstream sha256 are the trust
  anchors. Set `rke2_airgapped_artifacts_dir` to pin locally vetted artifacts.
- **Air-gap install:** you stage `rke2.linux-<arch>.tar.gz` +
  `sha256sum-<arch>.txt` yourself; the installer verifies the tarball checksum
  before extraction. Verify the checksum file's provenance out of band.

There is intentionally **no** binary signature (GPG/cosign) check; supply-chain
trust rests on TLS, the upstream sha256 manifest, and your air-gap staging.

## Ansible Vault

All sensitive variables should be encrypted using Ansible Vault:

```bash
ansible-vault encrypt_string 'mysecret' --name 'rke2_keepalived_auth_pass'
```

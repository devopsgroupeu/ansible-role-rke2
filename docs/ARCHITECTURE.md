# Architecture

This document describes how the role deploys RKE2 in HA mode and how it integrates
with external components (HAProxy/keepalived, HashiCorp Vault).

See also: [INTEGRATION.md](INTEGRATION.md)

---

## HA Control-Plane Topology

Three server nodes form an etcd quorum. kube-vip runs as a DaemonSet on every
control-plane node and uses ARP leader election to own the VIP. Agent nodes join
through the VIP address.

```mermaid
flowchart TD
    clients(["kubectl / API clients"])
    vip["VIP :6443\nkube-vip leader election"]

    subgraph cp["Control Plane — server_nodes"]
        s1["server1\nbootstrap"]
        s2["server2"]
        s3["server3"]
        etcd["etcd quorum\n3-node Raft"]
    end

    subgraph workers["Worker nodes — agent_nodes"]
        a1["agent1"]
        a2["agent2"]
    end

    clients -->|HTTPS 6443| vip
    vip --> s1
    vip --> s2
    vip --> s3
    s1 <-->|"9345 (join)"| s2
    s1 <-->|"9345 (join)"| s3
    s2 <-->|"9345 (join)"| s3
    s1 --- etcd
    s2 --- etcd
    s3 --- etcd
    a1 -->|"9345 (join) → VIP"| vip
    a2 -->|"9345 (join) → VIP"| vip
```

---

## External HAProxy VIP Pattern

When `rke2_vip_manager: kubevip` is not suitable (e.g. ARP/GARP is restricted on
the network), an external HAProxy+keepalived VIP can front the cluster. HAProxy runs
on `proxy_hosts` nodes and load-balances ports 6443 and 9345 across `server_nodes`.

```mermaid
flowchart TD
    clients(["kubectl / API clients"])

    subgraph lb["Load balancers — proxy_hosts"]
        hap1["haproxy1\nkeepalived MASTER"]
        hap2["haproxy2\nkeepalived BACKUP"]
        vip_ext["External VIP :6443/:9345\nkeepalived VRRP"]
    end

    subgraph cp["Control Plane — server_nodes"]
        s1["server1"]
        s2["server2"]
        s3["server3"]
    end

    subgraph workers["Worker nodes — agent_nodes"]
        a1["agent1"]
        a2["agent2"]
    end

    clients -->|HTTPS 6443| vip_ext
    vip_ext --> hap1
    hap1 <-->|VRRP| hap2
    hap1 -->|"6443 / 9345"| s1
    hap1 -->|"6443 / 9345"| s2
    hap1 -->|"6443 / 9345"| s3
    a1 -->|"9345 → VIP"| vip_ext
    a2 -->|"9345 → VIP"| vip_ext
```

In this pattern set `rke2_vip_enabled: false` (HAProxy manages the VIP externally)
and `rke2_agent_server_address` to the HAProxy VIP address.

---

## HA Bootstrap Sequence

```mermaid
sequenceDiagram
    participant A as Ansible Controller
    participant S1 as server1 (bootstrap)
    participant S2 as server2
    participant S3 as server3
    participant AG as agent1

    A->>S1: Install RKE2, write config (no server URL)
    A->>S1: Start rke2-server
    A->>S1: wait_for node-token (timeout 300 s)
    A->>S1: Fetch node-token via slurp
    A->>S1: wait_for port 9345 (cluster join API ready)

    A->>S2: Install RKE2, config (server: S1:9345, token)
    A->>S3: Install RKE2, config (server: S1:9345, token)
    A->>S2: Start rke2-server (join)
    A->>S3: Start rke2-server (join)

    A->>AG: Install RKE2, agent config (server: VIP:9345)
    A->>AG: Start rke2-agent

    A->>S1: Fetch kubeconfig → controller (if rke2_kubeconfig_download: true)
```

---

## Stack Composition Order

This role sits in the config tier, after network infrastructure is provisioned:

```
infrastructure (provision hosts: any cloud or bare-metal)
  → haproxy/keepalived (VIP layer)
    → hashicorp-vault (Raft cluster behind VIP)
      → rke2 servers (bootstrap + join)
        → rke2 agents (join via VIP)
          → in-cluster ESO / Vault Agent
```

Inventory group mapping:

| Group | Role that reads it |
|---|---|
| `server_nodes` | `devopsgroupeu.rke2` (control plane) |
| `agent_nodes` | `devopsgroupeu.rke2` (workers) |
| `proxy_hosts` | `devopsgroupeu.haproxy-keepalived` |
| `vault` | `devopsgroupeu.hashicorp-vault` |

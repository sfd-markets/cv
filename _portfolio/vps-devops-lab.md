---
title: "VPS DevOps Practice Lab"
excerpt: "Hetzner CX53 platform lab — Kind Kubernetes, Vault Helm HA, Terraform hcloud, and Ansible on Debian/AlmaLinux LXC."
collection: portfolio

---

## Intended Role

Hands-on portfolio project targeting **Platform Engineering**, **Cloud Infrastructure**, and **Site Reliability Engineering** roles. The lab documents end-to-end design of a production-style cloud platform on a **Hetzner Cloud CX53** VPS — a standalone, resource-efficient environment for certification prep and platform engineering practice.

| Focus Area | What the Lab Covers |
| :--- | :--- |
| **Cloud platform design** | Hetzner CX53 sizing, Cloud Firewall, optional volumes and DNS via Terraform |
| **Kubernetes platform ops** | Kind cluster (1 control-plane + 3 workers) with ingress-nginx, workload scheduling, and PVCs |
| **Infrastructure automation** | Ansible configuration management on heterogeneous Debian and AlmaLinux LXC targets |
| **Security & secrets engineering** | Edge firewall restricted to `HOME_IP/32`, Vault Helm HA in-cluster, host `vault-ops` Docker for Ops Pro drills |
| **IaC** | Terraform on host with `hcloud` provider for declarative Hetzner resource management |

## What This Project Demonstrates

- **Kind over nested kubelet** — Kubernetes workloads run in a Docker-backed Kind cluster instead of k3s-in-LXC, avoiding nested kubelet complexity and lowering RAM footprint
- **Vault in-cluster + host drills** — Vault Helm chart (Raft HA, Agent Injector, CSI) on worker 1; standalone `vault-ops` Docker on host for Vault Operations Professional exercises
- **Terraform on host** — `hcloud` provider and state on the CX53 host avoids bootstrap loops; manages firewall rules, volumes, and optional infrastructure
- **Heterogeneous Ansible fleet** — Debian 12 minimal control plus Debian and AlmaLinux 9 minimal targets on `lxdbr0` for RHCE-style `apt` / `dnf` playbooks
- **Zero-trust edge** — Hetzner Cloud Firewall and UFW allow SSH `:22` and Kubernetes API `:6443` from home `/32` only; Ansible SSH stays isolated on `lxdbr0`, separate from Kind CNI

## Architecture

| Component | Role |
| :--- | :--- |
| **cx53-host (Hetzner CX53)** | Debian 12 minimal; Docker, Kind, kubectl, helm, Terraform, LXD, UFW |
| **kind-control-plane** | API server, etcd, scheduler — no user pods |
| **kind-worker-1** | Vault HA (Helm), Agent Injector, CSI |
| **kind-worker-2** | CKAD lab pods, ingress-nginx |
| **kind-worker-3** | vault-demo namespace, PostgreSQL, demo API |
| **ansible-control (LXC)** | Ansible control node on `lxdbr0` |
| **ansible-target-1 (LXC)** | Debian 12 minimal RHCE target |
| **ansible-target-2 (LXC)** | AlmaLinux 9 minimal Red Hat family RHCE target |
| **vault-ops (Docker)** | Standalone Vault Ops Pro drills on host |

### Network Model

| Layer | Endpoint | Purpose |
| :--- | :--- | :--- |
| Public edge | `<CX53_PUBLIC_IP>:22`, `:6443` | SSH and `kubectl` from laptop (`HOME_IP/32` only) |
| `lxdbr0` bridge | `10.47.0.0/24` (example) | Ansible SSH between control and targets |
| Kind pod CIDR | Managed by Kind | Pod-to-pod networking; isolated from `lxdbr0` |

## Tech Stack

Hetzner Cloud · Debian 12 · Docker · Kind · Kubernetes · Helm · ingress-nginx · HashiCorp Vault · HashiCorp Terraform · Ansible · LXD/LXC · `hcloud` CLI · kubectl

## Certification Alignment

| Certification | Primary components | Coverage |
| :--- | :--- | :--- |
| **CKAD** | Kind workers 2–3, ingress-nginx, `kubectl` | Full |
| **Vault Operations Professional** | Vault Helm HA + host `vault-ops` Docker | Full |
| **Vault + Kubernetes** | Injector, CSI, K8s auth | Full |
| **Terraform Associate** | Terraform + `hcloud` on host | Full |
| **RHCE** | Debian control + Debian/AlmaLinux targets | Full — heterogeneous `apt` / `dnf` fleet |

## Key Design Decisions

1. **Debian 12 minimal host** — leaner idle footprint than full Ubuntu Server cloud images
2. **Minimal LXC images** — smallest practical rootfs per Ansible role; ~25 GB LXD ZFS pool
3. **Kind workers at 2 vCPU each** — frees budget for Ansible control, targets, and included `vault-ops`
4. **Explicit resource budgets** — v2 documents allocated vs. remaining capacity on 16 vCPU / 32 GB / 320 GB CX53
5. **Standalone project** — no external legacy dependencies; v2 supersedes earlier Ubuntu-based design

## Status

**Current:** Architecture design v2 complete with topology, network layout, storage plan, resource budgets, and certification mapping.

**Documented:** Kind cluster layout, Vault Helm deployment model, Ansible inventory patterns, Hetzner firewall rules, and host workload placement.

**Planned:** Terraform modules for Hetzner resources and Ansible playbooks for automated lab provisioning.

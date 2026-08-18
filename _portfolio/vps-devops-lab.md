# Zero-Trust Practice Lab — Portfolio Overview

I am building a three-node zero-trust lab on Oracle Cloud Infrastructure as a **compact OKD cluster** (the open-source distribution that underlies Red Hat OpenShift). Identity, workloads, and Ansible planes live on separate nodes with pinned namespaces. **FX Signal Lab** is the flagship app: a CronJob pulls forex data using a Vault-injected API key, a batch Job computes VSA and Bollinger signals, PostgreSQL stores history, and FastAPI serves charts behind an OpenShift Route. Cluster admin uses OIDC; SSH to Ansible VMs is Boundary-brokered; east-west traffic uses Service Mesh mTLS. Instances stop when I am not practicing to control cost.

This is a **target-state design** for a personal lab — not a production environment. Implementation detail lives in [`vps_oci_3node_lab_okd_kubevirt_architecture-v1.md`](vps_oci_3node_lab_okd_kubevirt_architecture-v1.md).


## Skills this lab demonstrates

- **OKD / OpenShift** — compact three-node cluster, Routes, OAuth, Operators, CRI-O
- **KubeVirt + CDI** — real guest OS VMs (Debian 12 and AlmaLinux 9) for Ansible practice
- **HashiCorp stack** — Terraform (`oci` provider), Vault HA, Boundary, Packer guest images
- **Ansible** — heterogeneous Linux fleet on KubeVirt, not containers pretending to be VMs
- **Zero-trust patterns** — OIDC for humans, Vault for secrets, Service Mesh mTLS for services
- **Application platform** — Python/FastAPI pipeline with CronJobs, Jobs, PVCs, and an external API


## Platform at a glance

Three AMD x86 `VM.Standard.E4.Flex` instances (4 OCPU / 32 GB / 200 GB each) in one OCI VCN form a single compact OKD cluster. Control-plane components run on all three nodes; **user** workloads are pinned with `lab.plane=identity|workloads|ansible`.

| Plane | Node | Role |
| ----- | ---- | ---- |
| **Identity & trust** | `oci-identity` | Secrets, identity, and mesh control — Vault, Boundary controller, Keycloak (OIDC), istiod |
| **Applications** | `oci-workloads` | FX Signal Lab — fetcher, analyzer, API, PostgreSQL, mesh sidecars |
| **Automation fleet** | `oci-ansible` | KubeVirt guests (Debian control + Debian/Alma targets) and a Boundary worker |

This split mirrors production thinking: security-critical services stay off the application blast radius, and the Ansible practice fleet stays off application pods.


## Mindmap

```mermaid
mindmap
  root((ZT_Practice_Lab))
    Why
      Practice_OKD
      Practice_KubeVirt_Ansible
      Portfolio_FX_Signal_Lab
      Demonstrate_zero_trust
    Identity
      Vault_HA
      Boundary_controller
      Keycloak_OIDC
      istiod
    Workloads
      fx_fetcher
      fx_analyzer
      fx_api
      PostgreSQL
      Mesh_sidecars
    Ansible
      KubeVirt_CDI
      Debian_control
      Debian_and_Alma_targets
      Boundary_worker
    Shared_OKD
      Compact_cluster
      FCOS_Ignition
      Routes
    ZT_triangle
      Vault_secrets
      Boundary_plus_OIDC
      Service_Mesh_mTLS
```


## Topology

North-south: my laptop authenticates to **OKD OAuth** (OIDC) for cluster admin and to **Boundary** for SSH into KubeVirt guests. FX Signal Lab is reached via an OpenShift Route. Public SSH is closed; API, OAuth, and Boundary are limited to a home IP.

East-west: FX pods pull secrets from Vault on the identity node over the private VCN. Mesh policies on identity apply to sidecars on workloads. Ansible SSH never crosses into `fx-lab` — guests are scheduled only on the ansible node.

Build-time: Terraform provisions the three instances and VCN. OKD installs on Fedora CoreOS via Ignition. Packer builds guest disks that CDI/KubeVirt consumes.

```mermaid
flowchart TB
  laptop[Laptop_WSL]

  subgraph oci [OCI_VCN]
    subgraph id [oci_identity]
      oauth[OKD_OAuth]
      vault[Vault_HA]
      boundary[Boundary_controller]
      meshC[istiod]
    end

    subgraph wl [oci_workloads]
      fx[fx_lab]
      route[Route]
      pg[(PostgreSQL)]
      meshD[Mesh_sidecars]
    end

    subgraph an [oci_ansible]
      kv[KubeVirt]
      ctrl[ansible_control_VM]
      targets[Debian_and_Alma_VMs]
      bW[Boundary_worker]
    end
  end

  laptop -->|"OIDC plus oc"| oauth
  laptop -->|"Boundary"| boundary
  laptop -->|"browser"| route

  boundary --> bW
  bW --> ctrl
  bW --> targets

  vault -->|"inject secrets"| fx
  meshC -->|"policies"| meshD
  fx --> meshD
  meshD -->|"mTLS fx-api only"| pg
  route --> fx
  ctrl -->|"Ansible SSH"| targets
```


## FX Signal Lab

A bounded forex analytics platform — not a throwaway nginx demo. It shows batch processing, persistent data, external API integration, and zero-trust secret handling on OKD.

| Component | Kind | Responsibility |
| --------- | ---- | -------------- |
| **fx-fetcher** | CronJob | Pull 30-minute OHLCV bars from Polygon.io onto a shared PVC |
| **fx-analyzer** | Job | Volume Spread Analysis and Bollinger Band signals → PostgreSQL and Plotly charts |
| **fx-api** | Deployment | FastAPI — health, signals, and charts |
| **postgresql** | StatefulSet | Signal history |
| **Route** | OpenShift Route | `fx.<lab-domain>` → `fx-api` |

```mermaid
flowchart LR
  subgraph external [External]
    polygon[Polygon.io API]
  end

  subgraph identity [oci_identity]
    vault[Vault HA]
    meshC[istiod]
  end

  subgraph fxLab [fx_lab namespace]
    cron[fx_fetcher CronJob]
    job[fx_analyzer Job]
    api[fx_api Deployment]
    pvc[(fx_data PVC)]
    pg[(PostgreSQL)]
    route[OpenShift Route]
    sidecar[Mesh sidecars]
  end

  vault -->|POLYGON_API_KEY| cron
  vault -->|POLYGON_API_KEY| api
  vault -->|dynamic DB creds| api
  cron -->|REST| polygon
  cron --> pvc
  job --> pvc
  job --> pg
  api --> sidecar
  sidecar -->|mTLS allow fx-api only| pg
  meshC --> sidecar
  route --> api
```


## Zero-trust model

Vault governs *what credentials exist*. Boundary plus OKD OAuth govern *who may access what*. The mesh governs *which services may communicate*.

| Pillar | Product | Lab implementation |
| ------ | ------- | ------------------ |
| **Verify explicitly** | OKD OAuth + Boundary + OIDC | Every admin session authenticated; no anonymous SSH |
| **Least privilege** | RBAC / SCC + Boundary + Vault | Scoped targets; ephemeral SSH credentials from Vault |
| **Assume breach** | Segregated planes | Vault on identity; apps on workloads; Ansible VMs on ansible |
| **Micro-segmentation** | OpenShift Service Mesh | mTLS between `fx-api` and PostgreSQL; default-deny policies |
| **Secrets** | Vault HA | API key and dynamic DB credentials — never in git |
| **Edge** | OCI NSGs | API / OAuth / Boundary from home IP only; public SSH denied |

```mermaid
flowchart TB
  subgraph human [Human_Access]
    user[User_Laptop]
    boundary[Boundary]
    oauth[OKD_OAuth]
    oidc[OIDC_Keycloak]
  end

  subgraph secrets [Secrets]
    vault[Vault_HA]
  end

  subgraph mesh [Service_Mesh]
    istiod[istiod]
    demoApi[fx_api]
    postgres[postgresql]
  end

  subgraph targets [Session_Targets]
    kvSSH[KubeVirt_guest_SSH]
    k8sAPI[OKD_API]
  end

  user -->|authenticate| oidc
  oidc --> oauth
  oidc --> boundary
  user -->|oc_login| oauth
  user -->|boundary_connect| boundary
  boundary -->|credential_library| vault
  boundary --> kvSSH
  oauth --> k8sAPI

  demoApi -->|mTLS_AuthorizationPolicy| postgres
  istiod --> demoApi
  istiod --> postgres
  demoApi -->|dynamic_DB_creds| vault
  demoApi -->|POLYGON_API_KEY| vault
```

Interview walkthrough: authenticate once, then secrets and service identity do the rest.

```mermaid
sequenceDiagram
  actor You
  participant OAuth as OKD_OAuth
  participant B as Boundary
  participant V as Vault
  participant FX as fx_fetcher
  participant Poly as Polygon_io
  participant A as fx_analyzer
  participant PG as PostgreSQL
  participant API as fx_api
  participant Mesh as Service_Mesh

  You->>OAuth: OIDC login
  You->>B: authenticate OIDC
  Note over You,B: Admin path - no standing SSH keys

  FX->>V: K8s auth plus inject POLYGON_API_KEY
  FX->>Poly: fetch OHLCV
  FX->>A: CSV on fx-data PVC
  A->>PG: write signals
  You->>API: GET charts via Route
  API->>V: dynamic DB creds
  API->>Mesh: connect to postgresql
  Mesh-->>PG: mTLS allow fx-api only
  API-->>You: Plotly HTML

  You->>B: connect ssh ansible-control
  Note over B: Worker on ansible node reaches KubeVirt VMs
```


## How it is built

```mermaid
flowchart TB
  subgraph buildTime [Build_Time]
    packer[Packer_guest_qcow2]
    terraform[Terraform]
    ignition[FCOS_Ignition_OKD]
    packer --> cdi[CDI_DataVolumes]
    terraform --> ignition
  end

  subgraph runTime [Runtime_ZeroTrust]
    vault[Vault]
    boundary[Boundary]
    mesh[Service_Mesh]
  end

  ignition --> runTime
  cdi --> runTime
```

| Stage | Tool | Delivers |
| ----- | ---- | -------- |
| **Infrastructure** | Terraform | Three Flex instances, VCN, NSGs |
| **Cluster** | OKD + Ignition | Compact three-node Fedora CoreOS cluster |
| **Guest images** | Packer | Debian 12 and AlmaLinux 9 qcow2 → CDI |
| **Configuration** | Operators / Helm / Ansible | Vault, Boundary, mesh, `fx-lab`, KubeVirt VMs |


## Inputs to outcomes

What has to exist, and what I can show.

```mermaid
flowchart LR
  subgraph inputs [Inputs]
    homeIP[Home_IP]
    tfCreds[OCI_credentials]
    oidc[OIDC_config]
    secret[POLYGON_API_KEY]
    images[Packer_guest_images]
  end

  subgraph platform [Platform]
    okd[Compact_OKD]
    zt[Vault_Boundary_Mesh]
  end

  subgraph outputs [Demo_outputs]
    o1[OIDC_oc_login]
    o2[Vault_injected_key]
    o3[Charts_on_Route]
    o4[Mesh_denies_non_fx_api]
    o5[Boundary_SSH_to_VMs]
  end

  homeIP --> okd
  tfCreds --> okd
  oidc --> zt
  secret --> zt
  images --> okd
  okd --> zt
  zt --> o1
  zt --> o2
  zt --> o3
  zt --> o4
  zt --> o5
```


## Technology stack

| Technology | Where | What it shows |
| ---------- | ----- | ------------- |
| **OKD** | All three nodes | Compact cluster, Routes, OAuth, Operators |
| **FX Signal Lab** | Workloads hub | CronJobs, Jobs, FastAPI, PostgreSQL, Polygon.io |
| **KubeVirt / CDI** | Ansible hub | Debian + AlmaLinux Ansible fleet |
| **Vault** | Identity hub | HA, Kubernetes auth, dynamic DB credentials |
| **Boundary** | Controller on identity; worker on ansible | Identity-brokered SSH — no standing host keys |
| **Service Mesh** | istiod on identity; sidecars on workloads | mTLS between `fx-api` and PostgreSQL |
| **Terraform** | Identity plane | Instances, VCN, NSGs |
| **Packer** | Laptop | Guest qcow2 for KubeVirt |
| **Ansible** | KubeVirt VMs | Heterogeneous `apt` vs `dnf` automation |


## Honest constraints

- **Paid x86 Flex, not Always Free Ampere.** Compact OKD needs comparable RAM on every node; ARM Always Free shapes are out of scope.
- **etcd spans all three nodes.** Plane isolation is for *user* workloads. Losing any node still affects control-plane quorum.
- **Stop-when-idle.** Compute bills while instances run; block volumes bill always-on. Cold start after stop is slower than a Kind lab.
- **Nested KVM.** KubeVirt needs hardware virtualization on the ansible node; emulation is not the daily path.

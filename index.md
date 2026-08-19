---
layout: default
title: Carlos
permalink: /
redirect_from:
  - /cv/
  - /cv/cv/
---

# Carlos

*Currently building a hands-on VPS cloud platform lab — see [Projects]({{ "/portfolio/" | relative_url }}).*
---

## Professional Summary

Infrastructure Reliability Specialist who builds, manages, secures, and operates enterprise VMware private cloud: ESXi hypervisors, vCenter, virtual servers, vSAN/HCI, and the observability stack. Runs ITIL Incident, Problem, and Change Enablement as a loop: restore service on-call via ServiceNow, eliminate root causes through RCA, then implement permanent fixes under change control (CAB in the current role). Defines SLO/SLA frameworks through Service Level Indicators (SLIs) and Service Level Objectives (SLOs) that reduced operational toil (~30%), builds telemetry with Splunk, Prometheus, and Grafana, deploys Kubernetes on virtualized infrastructure, automates with Ansible and PowerCLI, hardens ESXi to CIS/NIST, and delivered +25% data center savings through physical-to-virtual migration.

---

## Technical Skills

**Observability & Reliability:** Splunk, Prometheus, Grafana, metrics, logging, telemetry, Service Level Indicators (SLIs), Service Level Objectives (SLOs), SLO/SLA frameworks, Dynatrace
**IT Service Management:** ITIL Incident Management, Problem Management, Change Enablement (Change Management), ServiceNow, on-call, major-incident restore, RCA, Change Advisory Board (CAB)
**Virtualization:** ESXi/vSphere, vCenter, HA, DRS, vMotion, vSAN/HCI, VM lifecycle (provision, rightsize, decommission), Dell racadm, OpenManage Enterprise (OME), ESXi/vCenter patching, firmware lifecycle
**Cloud & Infrastructure:** Kubernetes, container orchestration, Linux, private cloud, hyper-converged infrastructure, distributed systems, Persistent Volumes, Storage Classes
**Automation & IaC:** Ansible, PowerCLI, PowerShell, playbook deployment, infrastructure automation, CI/CD pipelines
**Security & Compliance:** IAM, PAM, CIS/NIST compliance, ESXi hardening, encryption programs
**Networking & Tools:** Wireshark, strace, ServiceNow

---

## Professional Experience

### SRE Infrastructure Private Cloud Virtualization Lead **2022–Present**

Built, managed, secured, and operated the vSphere stack (ESXi, vCenter, virtual servers, vSAN/HCI) and the observability program for private cloud.

- On-call Incident Management via ServiceNow: log outages, apply immediate workarounds, restore service, and close tickets; Splunk dashboards for live visibility to siloed programs and senior management while incidents are open
- Problem Management after restore: correlate Prometheus and Grafana telemetry across vSAN/HCI compute and storage, then engage network, storage, application, and security teams to investigate RCAs for incidents and performance issues on the virtualization plant; document known errors and workarounds so the same faults do not return
- Rack-build and operate ESXi on Dell servers using racadm and OpenManage Enterprise (OME); operate vCenter inventory and clusters with HA, DRS, and vMotion
- Change Enablement for the virtualization stack: risk-assess ESXi, vCenter, and Dell firmware changes with maintenance windows and rollback; own vSAN/HCI health, storage policies, and VM lifecycle (provision, rightsize, reclaim). Member of the Change Advisory Board (CAB), providing IT and operational approval for engineering and operations changes
- Adopted Ansible, Python and PowerCLI within IT operations to automate VM and host tasks, reduce TOIL, and support CI/CD pipeline workflows
- Led IAM/PAM for Virtualization Operations, scoping entitlements to administer the virtual infrastructure privileged-access environment; implemented ESXi hardening and vAppliances (Proxy, DLP, AV); CIS/NIST audits
- Managed employees, projects, and day-to-day private cloud IT operations while partnering with engineering leaders and stakeholders across siloed programs to drive org-wide reliability improvements

### Core Infrastructure Senior Associate **2018–2021**

Built the virtualized plant and reliability model (SLOs, P2V, Kubernetes storage, encryption) later operated at Lead scope.

- On-call Incident Management via ServiceNow: restore service first with workarounds, then close incidents once the virtualization plant was stable
- Problem Management on recurring resource constraints (high CPU, memory contention, I/O bottlenecks): developed SLIs/SLOs to improve monitoring coverage, ran cross-team RCA on incidents and performance issues, reduced TOIL, and realized ~30% operational savings
- Change Enablement for physical-to-virtual migration: risk-assessed staged cutovers and performance benchmarks, saving +25% in data center space with less unplanned maintenance
- Configured Kubernetes storage using Persistent Volumes and Storage Classes on the virtualized platform to meet application requirements in production
- Ran the encryption program keeping the virtual data center (hosts, VMs, and storage components) compliant with security requirements

### Information Technology Infrastructure Consultant **2015–2018**

Managed Linux and hypervisor estates and built out virtualized storage and compute for enterprise operations.

- Managed and maintained a fleet of Linux servers and hypervisors: day-2 operations, configuration, and availability for enterprise infrastructure
- Built out virtualized storage and compute stacks, improving performance, efficiency, and scalability across the datacenter
- Delivered training, presentations, and reports to Core Infrastructure teams and stakeholders, documenting practices and raising operational preparedness

### NOC Systems Administrator Managed Services **2013–2015**

Operated production customer infrastructure: incident response, RCA, and availability under load.

- Performed root cause analysis (RCA) on production incidents by analyzing log data and system telemetry using strace, Dynatrace, and Wireshark across compute and network layers, supporting incident response for customer infrastructure
- Responded to high-volume events (e.g. Black Friday) by identifying systems under load and dynamically scaling customer environments to maintain availability across distributed systems
- Performed system administration including installation, configuration, and maintenance of Linux servers, network devices, and applications

---

## Education

### Bachelor of Arts - Completed

---

## Certifications

- **Certified Kubernetes Administrator (CKA)** — Linux Foundation (Oct 2025)
- **Certified Cloud Security Professional (CCSP)** — ISC2 (Jun 2024)
- **Offensive Security Certified Professional (OSCP)** — OffSec (Sep 2023)

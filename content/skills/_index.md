---
title: "Technical Skills"
date: 2025-05-19T14:56:00+01:00
draft: false
description: "DevOps, cloud-native, and multi-cloud engineering skills covering AWS, Azure, Google Cloud, and Alibaba Cloud, Kubernetes, Terraform, CI/CD, DevSecOps, and observability."
skill_categories:
  - name: "Cloud Platforms (Multi-Cloud)"
    skills:
      - name: "Amazon Web Services (AWS)"
        level: 90
        years: "4+"
        description: "EC2, S3, VPC, IAM, EKS, RDS, Lambda, CloudWatch"
      - name: "Microsoft Azure"
        level: 90
        years: "4+"
        description: "VMs, VNet, AKS, ACR, Azure DevOps, Monitor, Key Vault"
      - name: "Google Cloud Platform"
        level: 80
        years: "3+"
        description: "GCE, GKE, Cloud Storage, Cloud Build, Cloud Monitoring"
      - name: "Alibaba Cloud"
        level: 75
        years: "2+"
        description: "ECS, VPC, SLB, ACK, OSS, Container Registry"
  - name: "Containers & Orchestration"
    skills:
      - name: "Kubernetes"
        level: 90
        years: "4+"
        description: "Cluster lifecycle, ingress, autoscaling, Helm, operators"
      - name: "Docker"
        level: 92
        years: "5+"
        description: "Multi-stage builds, image hardening, registry management"
      - name: "Helm & Kustomize"
        level: 85
        years: "3+"
        description: "Packaging and templated deployments across environments"
      - name: "Service Mesh & Ingress"
        level: 78
        years: "3+"
        description: "NGINX Ingress, Istio, traffic routing, mTLS"
  - name: "Infrastructure as Code & Automation"
    skills:
      - name: "Terraform"
        level: 90
        years: "4+"
        description: "Multi-cloud provisioning, modules, remote state, workspaces"
      - name: "Ansible"
        level: 85
        years: "5+"
        description: "Configuration management, playbook automation, fleet rollout"
      - name: "Shell / Bash Scripting"
        level: 90
        years: "6+"
        description: "Automation, operational tooling, CI task scripting"
      - name: "Python"
        level: 82
        years: "5+"
        description: "Automation, API integrations, observability tooling"
  - name: "CI/CD & DevSecOps"
    skills:
      - name: "CI/CD Pipelines"
        level: 92
        years: "5+"
        description: "GitLab CI, Jenkins, GitHub Actions, Azure Pipelines"
      - name: "Azure DevOps"
        level: 88
        years: "4+"
        description: "Pipelines, boards, repos, artifact feeds, release gates"
      - name: "DevSecOps & Security Scanning"
        level: 85
        years: "3+"
        description: "SAST/DAST, image scanning, secret management, policy as code"
      - name: "Git & Branching Strategy"
        level: 90
        years: "6+"
        description: "Trunk-based and GitFlow, code review discipline, release tagging"
  - name: "Observability & Reliability"
    skills:
      - name: "OpenTelemetry & OpAMP"
        level: 90
        years: "3+"
        description: "Collector fleet management, auto-instrumentation, OTLP pipelines"
      - name: "Prometheus & Grafana"
        level: 90
        years: "4+"
        description: "Metrics collection, dashboards, alerting, SLO tracking"
      - name: "Centralised Logging"
        level: 85
        years: "4+"
        description: "Log aggregation and search pipelines (ELK / Loki style stacks)"
      - name: "Incident Response & SRE Practice"
        level: 85
        years: "3+"
        description: "On-call, runbooks, blameless post-mortems, RTO/RPO planning"
  - name: "Data Centre & Platform Engineering"
    skills:
      - name: "Linux Administration"
        level: 92
        years: "8+"
        description: "RHEL/Ubuntu hardening, systemd, networking, performance tuning"
      - name: "Networking & Load Balancing"
        level: 85
        years: "6+"
        description: "DNS, firewalls, reverse proxies, HA load balancing"
      - name: "Virtualisation & Backup/DR"
        level: 82
        years: "5+"
        description: "VMware/KVM, capacity planning, backup strategy, disaster recovery"
      - name: "Database Operations"
        level: 80
        years: "5+"
        description: "MySQL/PostgreSQL administration, replication, backup automation"
  - name: "Application Development Background"
    skills:
      - name: "Backend Development"
        level: 85
        years: "6+"
        description: "REST APIs, authentication, service design (PHP, Python, Node.js)"
      - name: "Frontend Development"
        level: 80
        years: "5+"
        description: "Responsive web UI, modern JavaScript frameworks"
      - name: "System Design"
        level: 82
        years: "4+"
        description: "High-availability architecture, scalability, cost-aware design"
---

Platform engineering is only useful if it makes the people building on it faster and safer. My background started in application development, moved into infrastructure and automation, and now focuses on running cloud-native platforms at enterprise scale.

I work across **four major clouds** — AWS, Azure, Google Cloud, and Alibaba Cloud — choosing services based on cost, compliance, and workload fit rather than familiarity. Everything I provision is described as code, every change flows through a reviewed pipeline, and every service is instrumented so that health questions are answered with data instead of guesswork.

## Recent focus

### OpAMP control panel for OpenTelemetry Collector fleets
I am currently implementing an **OpAMP (Open Agent Management Protocol) control panel** for the fleet management of **OpenTelemetry Collectors**. Rather than installing and configuring observability agents host by host, collectors register with a central control plane, receive their configuration remotely, and can be updated, audited, and rotated at fleet scale.

The outcome is **automatic observability agent setup**: new services and hosts are onboarded to full metrics, logs, and traces without manual installation steps, which removes a recurring source of toil and configuration drift.

### DevSecOps across the delivery pipeline
Security is built into the pipeline rather than bolted on afterwards — dependency and image scanning, secret management, and policy checks all run as part of the standard build, so the secure path is also the easiest path.

### Data centre operations at enterprise scale
Alongside cloud platform work, I manage enterprise data centre operations: capacity planning, networking, virtualisation, backup and disaster recovery, and hardware lifecycle management.

## Certifications

- **Microsoft Certified: Azure Administrator Associate**
- **Microsoft Certified: Azure DevOps Engineer Expert**
- **Microsoft Certified: Azure Fundamentals**
- **Microsoft Certified: Azure Data Fundamentals**

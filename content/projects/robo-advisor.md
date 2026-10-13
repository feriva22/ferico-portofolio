---
date: '2024-08-25T09:53:42+02:00' # date in which the content is created - defaults to "today"
title: 'Observability at Fleet Scale with OpAMP'
draft: false # set to "true" if you want to hide the content 

link: "https://www.linkedin.com/in/ferico" # optional URL to link the logo to

params:
    button:
        icon: "icon-arrow-right"
        btnText: "View my experience"
        URL: "/experience"
    image:
        src: "images/works/robo-advisor.jpg"
        scale: 0.5
    
## The content is used for the description of the project
---

An **OpAMP (Open Agent Management Protocol) control panel** for the **fleet management of OpenTelemetry Collectors** — the initiative I am currently leading.

Traditionally, onboarding a host or service into observability meant installing a collector, writing its configuration locally, and repeating that work for every new node. At fleet scale that becomes toil, drift, and blind spots.

With the OpAMP control panel, collectors register with a central control plane and receive their configuration remotely. Configuration changes are rolled out centrally, agent versions are tracked, and unhealthy collectors are visible instead of silently failing. The outcome is **automatic observability agent setup**: metrics, logs, and traces are enabled consistently across environments without manual per-host work.

This is paired with **Prometheus** and **Grafana** dashboards, alerting tied to SLOs, and distributed tracing through the **OpenTelemetry** pipeline.
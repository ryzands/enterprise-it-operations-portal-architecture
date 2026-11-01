# Enterprise IT Operations Portal & Hybrid Storage Architecture
**Sanitized Architectural Case Study | Full-Stack Engineering & Infrastructure Cost Optimization**

> **Disclaimer:** *This repository documents the architectural design, infrastructure topology, and engineering decisions behind an internal enterprise system deployed across a multi-branch banking institution. To comply with strict banking regulations and Non-Disclosure Agreements (NDAs), proprietary source code, internal IP addresses, and company-specific branding have been sanitized.*

---

## 1. Executive Summary & The Product Realization

In a multi-branch financial institution (10+ branch offices), IT operations were previously fragmented across scattered spreadsheets, chat groups, and a legacy intranet. 

When designing the modern replacement (**IT Hub**), initial assumptions focused heavily on building a complex ticketing engine (SLAs, assignments, queues). However, analyzing actual user behavior from the legacy system revealed a critical product truth: **employees did not visit the portal because they wanted to open tickets; they visited to find operational information, application links, SOPs, forms, and PIC contacts.**

### Strategic Shift & Business Impact
* **Discovery-First Architecture:** Re-architected the platform from a passive "ticket dropbox" into an active **Operational Knowledge & Self-Service Hub**. By surfacing SOPs, quick links, and guided troubleshooting *before* ticket creation, routine support inquiries were deflected at the source.
* **60%+ Cloud Storage Cost Reduction:** Implemented a hybrid cloud-on-premise object storage pipeline. Short-lived, high-volume user uploads (diagnostic screenshots, temporary logs) are routed to on-premise **MinIO / TrueNAS** instances over an encrypted mesh network, preventing cloud storage bloat.
* **Zero-License Overhead:** Replaced potential third-party SaaS costs (Jira Service Management / Confluence per-seat pricing) with a fully self-hosted, banking-compliant stack.

---

## 2. System & Hybrid Storage Architecture

The system operates on a split-horizon hybrid architecture. The core web application remains highly available, while heavy object storage leverages on-premise NAS hardware securely exposed via a private WireGuard/Tailscale mesh tunnel and pfSense routing.

```mermaid
graph TD
    subgraph "Branch Offices & Remote Users"
        U[Branch Employees / IT Staff]
    end

    subgraph "Application & Edge Layer (Virtualized Server)"
        NP[Nginx Reverse Proxy / SSL Termination]
        FE[Next.js App Router - Frontend Portal]
        BE[NestJS Modular API - Backend Engine]
        DB[(PostgreSQL - Core Relational Data)]
    end

    subgraph "Encrypted Hybrid Mesh (Tailscale / pfSense)"
        TS[Tailscale / WireGuard Tunnel]
    end

    subgraph "On-Premise Storage Tier (Cost Optimization)"
        MINIO[MinIO S3-Compatible Gateway]
        NAS[(TrueNAS ZFS Storage Pool)]
        BKP[Automated Rclone / Rsync Offsite Sync]
    end

    U -->|HTTPS| NP
    NP --> FE
    NP -->|/api/v1/*| BE
    FE -->|SSR / REST API| BE
    BE -->|Prisma ORM| DB
    BE -->|Pre-Signed S3 URLs via Mesh| TS
    TS --> MINIO
    MINIO --> NAS
    NAS -.->|Nightly Cron| BKP

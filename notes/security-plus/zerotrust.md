---
title: "Zero Trust Security Model"
tags: ["security", "zero-trust", "networking"]
prerequisites: []
status: "learning"
last_updated: "2025-10-24"
---

## Summary

Zero Trust rejects the idea of a trusted internal network perimeter. Every request is treated as hostile until it is explicitly authenticated, authorized, and encrypted. Following NIST SP 800-207, a Zero Trust program blends continuous verification, per-session access decisions, segmentation, least privilege, and automated response to limit the blast radius of any breach.

## Core Principles

### No Implicit Trust Zone
- **Description**: Neither users nor devices are trusted by default, no matter whether they originate from inside or outside the corporate network.
- **Key Takeaway**: The perimeter model is obsolete—trust must be earned for every interaction.

### No Ownership Assumption
- **Description**: Operate as if the organization does not own the connecting network or device, a realistic stance in BYOD and cloud contexts.
- **Key Takeaway**: Secure data and access paths rather than relying on presumed-safe infrastructure.

### Insecure Connections
- **Description**: Treat all network traffic as potentially hostile, requiring encryption, authentication, and continuous monitoring.
- **Key Takeaway**: Encrypt everything, everywhere, and instrument visibility.

### Consistent Security Policies
- **Description**: Apply policies uniformly across on-prem, cloud, and hybrid environments for every identity and device.
- **Key Takeaway**: Centralized, dynamic controls prevent policy drift and coverage gaps.

## Framework for Zero Trust

### Continuous Verification
- **Practice**: Always verify access, all the time, for every resource request—no implicit trust remains after initial login.
- **Implications**: Enforce strong identity, device posture checks, and context-aware authentication for each transaction.

### Access Limitation
- **Practice**: Grant access to individual enterprise resources on a per-session basis, scoped to the requested task.
- **Implications**: Apply least privilege, enforce short-lived tokens, and require reauthorization when context changes.

### Limit the Blast Radius
- **Practice**: Minimize impact if internal or external resources are breached through segmentation and least privilege.
- **Implications**: Design microsegments, monitor east-west traffic, and ensure workloads default to deny inter-segment communication.

### Automate
- **Practice**: Automate context collection and response across credentials, workloads, endpoints, SIEMs, and threat intelligence feeds.
- **Implications**: Use orchestration tooling to codify policy evaluation, trigger incident playbooks, and keep pace with attacker velocity.

---

## Zero Trust Architecture Components

### Plane
- **Control Plane**: The "brains" of the system. It makes access decisions based on policies and signals from various sources (identity providers, device health, etc.).
- **Data Plane**: The "muscle" of the system. It enforces the decisions made by the control plane.

### Key Components
- **Policy Decision Point (PDP)**: The control plane component that makes the access decision. Also known as the Policy Engine.
- **Policy Enforcement Point (PEP)**: The data plane component that enforces the policy (e.g., a gateway, proxy, or agent).
- **Policy Administrator (PA)**: The component responsible for creating and managing security policies.
- **Policy Engine (PE)**: The component that evaluates policies and makes decisions about whether to allow or deny access.
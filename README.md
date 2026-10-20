# Multi-Container Wazuh SIEM Deployment & Troubleshooting Guide

A comprehensive guide documenting the end-to-end deployment of a single-node Wazuh SIEM stack on Kali Linux via Docker Compose. This document walks through diagnosing system dependency issues, adjusting kernel constraints, correcting JVM security manager certificate permission locks, and achieving a successful production deployment.

> **Detailed Technical Documentation:** For the complete, step-by-step penetration testing and SIEM logging documentation, refer to the [Comprehensive Workflow Document](workflow.md).

---

## 📊 Executive Summary

This project documents the transition from a failed bare-metal/local package installation to a fully functional, containerized Wazuh SIEM stack on Kali Linux. By addressing underlying system service conflicts, host kernel limitations, and cryptographic asset permissions, the environment was successfully stabilized into a multi-container architecture.

### Key Highlights
* **Architectural Migration:** Shifted from local package deployment to Docker Compose containerization to ensure service isolation and dependency control.
* **Kernel Level Tuning:** Identified and resolved OpenSearch cluster boot failures by reconfiguring Linux kernel virtual memory map limits.
* **Java Security Compliance:** Remedied certificate read exceptions caused by POSIX permission mismatches across persistent volume mounts.
* **Release Pinning:** Established deployment stability by locking component image builds to verified release tags (`v4.9.0`).

---

## 🛠️ Deployment & Troubleshooting Roadmap

### 1. Initial Local Package Installation Failures
* **Service Conflicts:** Direct package installations failed due to systemctl control layer blocks, resulting in socket connection refusals on port `5601`.
* **Cryptographic Locks:** OpenSearch database indexer failed to bootstrap because the security plugin could not locate or read `/etc/wazuh-indexer/certs/root-ca.pem`.

### 2. Container Migration & Kernel Remediation
* **Environment Isolation:** Migrated to Docker Compose to decouple components into isolated single-purpose containers (`indexer`, `manager`, `dashboard`).
* **Kernel Restrictions:** Solved container crash loops caused by the default `vm.max_map_count` allocation being lower than the OpenSearch runtime threshold (`262144`).
* **Volume Purging:** Cleared stale volume caches and invalid path locks using `docker compose down -v`.

### 3. Stack Resolution & Health Verification
* **Permission Alignment:** Applied strict `1000:1000` UID/GID ownership and `750` folder permissions to satisfy Java Security Manager policies inside containers.
* **Version Locking:** Pinned `wazuh-docker` to tag `v4.9.0` to eliminate component version mismatches across the stack.
* **Verified Health:** Successfully initialized the 4-container stack into a `Healthy` / `Running` operational status.

---

## 🚀 Production Remediation Playbook

To replicate the stable, operational Wazuh SIEM environment in a clean workspace, execute the following steps:

### Step 1: Tune Host Linux Kernel
Increase the virtual memory map capacities to satisfy OpenSearch operational minimums:
```bash
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf

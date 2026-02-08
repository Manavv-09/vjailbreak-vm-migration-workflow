
# vJailbreak VM Migration Workflow  
**Case Study: VMware ESXi → OpenStack**

---

## 1. About the Tool

vjailbreak is an open-source migration tool developed by Platform9.  
It helps organizations migrate virtual machines from VMware environments to OpenStack-compatible clouds or PCD platforms with reduced complexity and downtime.

It is often referred to as a “VMware escape tool”.

---

## 2. Objective

This case study documents my learning and hands-on experience performing a virtual machine migration using the vjailbreak tool.

The objective was to understand the tool’s architecture and observe real-world infrastructure transition workflows, including:

- Appliance bootstrapping  
- Disk migration  
- Network and storage mapping  
- Post-migration validation  

This migration involved moving workloads from **VMware ESXi** to an **OpenStack** environment.

Rather than relying solely on documentation, I focused on analyzing internal workflows, identifying failure points, and troubleshooting issues encountered during the migration process.

---

## 3. High-Level Migration Overview

At a high level, the migration workflow operates as follows:

- Connects to the source VMware vCenter / ESXi environment to discover virtual machines  
- Allows selection of VMs to migrate  
- Converts VMware disk formats (VMDK) into OpenStack-compatible formats such as QCOW2 or RAW  
- Recreates the VMs in OpenStack / PCD with mapped CPU, RAM, network, and storage configurations  
- Automates the end-to-end migration workflow  

[Module 01 - vJailbreak VM](module-01-appliance.md)
---





				





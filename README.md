# Cloud Sizing Scripts

This repository contains PowerShell scripts for cloud resource discovery. These scripts are designed to assist Commvault representatives in gathering information about cloud resources that may need protection, and help representatives in estimating the cost of protecting these resources. For setup instructions and steps to run, refer to the individual script files in each cloud provider folder.

## AWS 
Script: `AWS/CVAWSCloudSizingScript.ps1` 
Discovers AWS cloud resources to assist with Commvault protection planning.

## Azure
Script: `Azure/CVAzureCloudSizingScript.ps1`
Discovers Azure cloud resources to assist with Commvault protection planning.

## Google Cloud
Script: `GoogleCloud/CVGoogleCloudSizingScript.ps1`
Discovers Google Cloud resources to assist with Commvault protection planning.

## OCI
Script: `OCI/CVOracleCloudSizingScript.py` 
Discovers OCI cloud resources to assist with Commvault protection planning.

---

## Security Hardening (this fork)

This fork is a security-hardened copy of the upstream Commvault cloud sizing
scripts. The upstream repository is preserved as the `upstream` git remote.
A pre-execution security review identified six High severity findings across
these scripts; the patches below address all six.

Maintainer: Wil Ramos;
Review date: 2026-04-24

### `OCI/CVOracleCloudSizingScript.py` &mdash; 3 High

- Added `_validate_ocid` (compartment / cluster OCID format check) and
  `_validate_region` (allowlist against the published OCI region set).
  Inputs that fail validation never reach the OCI SDK or the kubectl
  subprocess.
- Switched every `subprocess` call from a string command line to an argv
  list with `shell=False`, eliminating the shell re-parse path.
- The kubeconfig file is now created via `tempfile.mkstemp()` with mode
  `0o600` and removed in a `try/finally` block, instead of being written
  to a path derived from a (potentially attacker-controlled) cluster ID.

### `AWS/CVAWSCloudSizingScript.ps1` &mdash; 1 High

- Added `Process-EKSCluster` validators for cluster name, region, and the
  IAM role ARN before any `aws eks` CLI call.
- The kubeconfig is written to a per-process temporary file and deleted on
  function exit (mirrors the OCI fix above).

### `Azure/CVAzureCloudSizingScript.ps1` &mdash; 1 High

- Added validators inside `Get-AKSPersistentVolumeInfo` for
  `SubscriptionId`, `ResourceGroup`, and `ClusterName` so that an attacker
  cannot smuggle additional `az aks` CLI arguments via these parameters.

### `GoogleCloud/CVGoogleCloudSizingScript.ps1` &mdash; 1 High

- Added `Test-GcpProjectId` (RFC-compliant project-ID allowlist) and a
  hard-asserting `Assert-GcpProjectId` wrapper.
- Both the `Get-GcpProjects` listing path and the `-Projects` parameter
  path now validate every project ID before it is passed to the gcloud
  CLI.

### Verification

- `OCI/CVOracleCloudSizingScript.py`: `python -m py_compile` clean.
- All four patched PowerShell files: parse-tested under PowerShell 7.4.6.

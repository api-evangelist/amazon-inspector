# Amazon Inspector (amazon-inspector)
Amazon Inspector is an automated vulnerability management service that continually scans AWS workloads for software vulnerabilities and unintended network exposure, providing detailed findings and prioritized remediation guidance.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-inspector/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Compliance, Container Security, EC2, Lambda, Security, Vulnerability Scanning

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AWS Amazon Inspector API
The Amazon Inspector API provides programmatic access to vulnerability management for scanning EC2 instances, container images, and Lambda functions for software vulnerabilities and network exposure.

**Human URL:** [https://aws.amazon.com/inspector/](https://aws.amazon.com/inspector/)

#### Tags:

 - Compliance, Security, Vulnerability Scanning

#### Properties

- [Documentation](https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html)
- [OpenAPI](openapi/amazon-inspector-openapi-original.yml)
- [GettingStarted](https://aws.amazon.com/inspector/getting-started/)
- [Pricing](https://aws.amazon.com/inspector/pricing/)
- [FAQ](https://aws.amazon.com/inspector/faqs/)

## Common Properties

- [Portal](https://aws.amazon.com/inspector/)
- [Website](https://aws.amazon.com/inspector/)
- [Documentation](https://docs.aws.amazon.com/inspector/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/security/category/security-identity-compliance/amazon-inspector/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/inspector/v2/home)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)

## Features

| Name | Description |
|------|-------------|
| Automated Vulnerability Scanning | Continuously scans EC2, container images, and Lambda functions for software vulnerabilities. |
| Risk Scoring | Ranks vulnerabilities by exploitability and impact to prioritize remediation. |
| SBOM Export | Generates software bill of materials for scanned workloads. |
| Multi-Account Support | Manages vulnerability scanning across all accounts in an AWS Organization. |

## Use Cases

| Name | Description |
|------|-------------|
| CI/CD Security Scanning | Automatically scan container images in ECR during build pipelines. |
| Compliance Reporting | Generate vulnerability reports for SOC 2, PCI DSS compliance. |
| Patch Prioritization | Prioritize OS patches based on exploitability scores. |

## Integrations

| Name | Description |
|------|-------------|
| Amazon ECR | Automatically scans container images stored in Elastic Container Registry. |
| AWS Security Hub | Sends all findings to Security Hub for centralized visibility. |
| AWS Organizations | Manages Inspector across all organizational accounts. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [AWS Amazon Inspector API](openapi/amazon-inspector-openapi-original.yml)

### JSON Schema

200 schema files covering key resources and operations.

### JSON Structure

200 JSON Structure files converted from JSON Schema.

### JSON-LD

- [Amazon Inspector Context](json-ld/amazon-inspector-context.jsonld)

### Examples

200 example JSON files generated from schemas.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [AWS Amazon Inspector API](capabilities/shared/inspector.yaml) — operations for amazon inspector management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Security Vulnerability Management](capabilities/security-vulnerability-management.yaml) | Amazon Inspector | 8 | Security Engineer, Cloud Security Engineer |

## Vocabulary

- [Amazon Inspector Vocabulary](vocabulary/amazon-inspector-vocabulary.yaml) — Unified taxonomy mapping resources, actions, workflows, and personas

## Rules

- [Amazon Inspector Spectral Rules](rules/amazon-inspector-spectral-rules.yml) — 14 rules across 6 categories enforcing Amazon Inspector API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com

# Modernization Plan: Cloud Modernization

**Project**: PhotoAlbum-Java

---

## Technical Framework

- **Language**: Java 25
- **Framework**: Spring Boot 4.0
- **Build Tool**: Maven
- **Database**: Oracle Database Free (to be migrated to Azure Database for
  PostgreSQL)
- **Key Dependencies**: Spring MVC, Thymeleaf, Spring Data JPA, Hibernate,
  Spring Security, Oracle JDBC

The existing assessment describes the former Java 8/Spring Boot 2.7 baseline.
The repository has already been upgraded to Java 25 and Spring Boot 4.0; those
upgrade recommendations are therefore out of scope for this plan.

---

## Overview

This modernization moves the PhotoAlbum-Java application from its local
Oracle-and-Docker deployment model to a maintainable Azure-hosted architecture.
The application currently stores photo metadata and image BLOBs in Oracle and
uses environment-injected credentials for local containers. The new
architecture will:

- replace Oracle persistence with Azure Database for PostgreSQL while
  preserving photo upload, browsing, navigation, and deletion behavior;
- make runtime configuration and application operation suitable for a
  stateless cloud deployment; and
- provision the required Azure resources and deploy the application to Azure
  Container Apps.

The migration will establish a baseline, update the application and data
configuration, provision infrastructure, verify the migrated application,
apply security remediation, and then deploy it.

---

## Migration Impact Summary

| Application | Original Service | New Azure Service | Authentication | Comments |
|-------------|------------------|-------------------|-------------|----------|
| PhotoAlbum-Java | Oracle Database | Azure Database for PostgreSQL | Managed identity | Migrate JPA data and SQL |
| PhotoAlbum-Java | Docker Compose runtime | Azure Container Apps | Managed identity | Provision and deploy app |
| PhotoAlbum-Java | Local runtime config | Azure Key Vault/configuration | Managed identity | Keep secrets externalized |

---

## Proposed Azure Architecture

The containerized Spring Boot application runs in Azure Container Apps and
connects to Azure Database for PostgreSQL. Azure Key Vault stores secrets that
cannot use passwordless authentication. Managed identity is the default
authentication approach for Azure resources. Application logs remain available
through the Azure-hosted container environment.

---

## Scope and Success Criteria

- Oracle-specific persistence behavior and SQL are compatible with PostgreSQL.
- Existing photo upload, retrieval, gallery navigation, and deletion flows
  continue to work after migration.
- Azure infrastructure is defined as Bicep and provisioned for the application.
- The application is built as a production container and deployed to Azure
  Container Apps.
- Configuration and secrets are externalized; no credentials are committed.
- The application builds, tests pass, integration verification succeeds, and
  dependency vulnerabilities are remediated before deployment.

---

## Open Questions & Questionnaire

- [x] Q: Should infrastructure be provisioned? → A: Yes, provision new Azure
  infrastructure using Bicep.
- [x] Q: Which deployment target should be used? → A: Azure Container Apps
  (the default target because no target was specified).
- [x] Q: Should integration testing be included? → A: Yes, use real provisioned
  Azure resources for post-migration verification.
- [x] Q: Should security/CVE remediation be included? → A: Yes, use the default
  dependency scan and remediation task.
- [ ] Azure subscription, region, and resource-group details were not
  provided; supply them when provisioning the generated infrastructure.


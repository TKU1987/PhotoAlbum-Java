# Infrastructure Plan: Photo Album Java Cloud Modernization

## User Requirements

Modernize the Java Photo Album application for Azure by addressing cloud
readiness, migrating its Oracle database dependency to Azure Database for
PostgreSQL, provisioning the required Azure resources, and deploying the
application to Azure Container Apps. Ignore runtime upgrade recommendations;
the application is already upgraded.

Use the existing resource group `rg-labuser-0006` in `swedencentral` and
subscription `381bd33f-79a1-4349-86e3-0eecd04338b1`. Do not create a resource
group.

**Plan Configuration**:

| Parameter | Value | Description |
|-----------|-------|-------------|
| IaC Tool | Bicep | Infrastructure as Code |
| Provision | true | Provision application resources during execution |
| Subscription | 381bd33f-79a1-4349-86e3-0eecd04338b1 | Target Azure subscription |
| Resource Group | rg-labuser-0006 | Existing resource group; do not create another |
| Location | swedencentral | Target Azure region |

## Proposed Architecture

```text
Existing resource group: rg-labuser-0006 (swedencentral)
  Azure Container Registry
        |
        | application image
        v
  Azure Container Apps environment --- Log Analytics workspace
        |
        | private/restricted database access
        v
  Azure Database for PostgreSQL Flexible Server
        ^
        |
  Photo Album Java application
```

The application will be built into a container image and deployed to Azure
Container Apps. The Oracle dependency currently declared by the application
will be migrated to Azure Database for PostgreSQL. The existing
`azure-setup.ps1` provisions AKS, chooses a random resource group in
`westus3`, and creates PostgreSQL resources; the target infrastructure must
instead use the supplied existing resource group and region and support
Container Apps. Do not provision AKS or create a resource group.

## Azure Resource List

| Resource Type | Resource Name | SKU | Est. Monthly Cost | Purpose |
|---------------|---------------|-----|------------------|---------|
| Azure Container Registry | Project-scoped unique name | Basic | Not estimated | Store the application image |
| Azure Container Apps managed environment | Photo Album environment | Consumption | Not estimated | Host the container application |
| Azure Container App | Photo Album application | Consumption | Not estimated | Run the Java web application |
| Azure Database for PostgreSQL Flexible Server | Project-scoped unique name | To be sized | Not estimated | Replace the Oracle database |
| Log Analytics workspace | Project-scoped unique name | Pay-as-you-go | Not estimated | Collect Container Apps logs |

Monthly costs are not estimated because no Azure pricing source or sizing
assumptions were provided. Resource names and database sizing should be
finalized during IaC generation while avoiding conflicts in the existing
resource group.

## Modernization Scope

The repository is a Java 25 / Spring Boot 4.0.9 Maven application. Oracle is
confirmed in its JDBC dependency and Spring datasource configuration, and the
Docker Compose setup starts an Oracle database. Existing Azure setup scripts
do not match the requested destination: they configure AKS and create a
separate resource group in another region. The modernization will:

- Move the application's database dependency from Oracle to Azure Database
  for PostgreSQL.
- Provision the Azure resources required for the application in the specified
  existing resource group and region.
- Deploy the application to Azure Container Apps, using the provisioned
  container registry and managed environment.
- Keep credentials out of source control and restrict database access to the
  application rather than using broad firewall access.

## Security Compliance

Scan project dependencies for known CVEs after the database transformation
and remediate identified vulnerabilities before deployment. Use the minimum
patched versions where possible; document any required major-version changes
and associated compatibility risks.

## Infrastructure Task

**Description**: Generate Bicep infrastructure for Azure Database for
PostgreSQL and Azure Container Apps, then provision it in the existing
`rg-labuser-0006` resource group in `swedencentral`.

**Output**: Validated Bicep files and provisioned Azure resources.

**Skill**: `infrastructure-bicep-generation`

**Success Criteria**:
- Bicep targets the provided subscription and existing resource group.
- No new resource group or AKS cluster is created.
- Required database and Container Apps resources are provisioned.

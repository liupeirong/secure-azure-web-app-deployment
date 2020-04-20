# ARM template that deploys the infrastructure to secure a web site on Azure

This example deploys the infrastructure for a web site that consists of the following components:
* A Single Page App hosted on Azure Blob Storage [Static Website](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website)
* A ASP.NET Core Web API hosted on Azure App Service
* A Azure SQL Database

The goal is to:
* Protect the database so that only the Web API can access it
* Protect the Web API so that only the Application Gateway can access it
* No secrets are stored in the Web API
* No certificates are stored in the Application Gateway 

![Alt text](infra.png?raw=true "infrastructure")

The ARM template implements the following security measures:
* [Azure SQL DB and Azure SQL Server firewalls](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-firewall-configure) are configured to only allow the Web API [outbound IPs](https://docs.microsoft.com/en-us/azure/app-service/overview-inbound-outbound-ips#find-outbound-ips) to access the database.
* Azure Web App accesses SQL DB and Blob Storage using [Managed Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) (MSI).
* [Azure Web App Access Restrictions](https://docs.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions) are configured to only allow Azure Application Gateway to access the Web API.
* Azure Application Gateway is configured for [end-to-end SSL](https://docs.microsoft.com/en-us/azure/application-gateway/end-to-end-ssl-portal) and [SQL-injection and XSS protection](https://docs.microsoft.com/en-us/azure/web-application-firewall/ag/ag-overview).
* Azure Application Gateway [retrieves SSL certificate from Key Vault](https://docs.microsoft.com/en-us/azure/application-gateway/configure-keyvault-ps) using MSI.

The ARM template also implements the following availability  measures:
* Application Gateway is deployed in [2 zones](https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-autoscaling-zone-redundant).
* Azure Blob Storage is configured for [Zone Redundancy](https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy) and [CDN](https://docs.microsoft.com/en-us/azure/cdn/cdn-create-a-storage-account-with-cdn).

Similar alternatives - 
* [Azure Front Door](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview) - can load balance across regions while Application Gateway can only load balance within a region. If there's no need for cross-region routing, Application Gateway has one IP address that can be used to lock down App Service. Front Door has a much wider range of outbound IP addresses. 
* [Azure SQL DB VNet Service Endpoints](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=/azure/virtual-network/toc.json) - restrict access to SQL from only a virtual network (VNET). However App Service can't be placed in a VNet without using [App Service Environment]().
* [Azure App Service Private Endpoints](https://docs.microsoft.com/en-us/azure/app-service/networking/private-endpoint) - restrict access to App Service from only a VNet. However this is currently (Apr 2020) in Preview only in 2 regions. 
* [Azure App Service Service Endpoints](https://docs.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions#service-endpoints) - restrict access to App Service from a VNET. You need to set up Service Endpoints on the VNET and Access Restrictions on App Service. If there's only one IP to lock down, setting Access Restrictions alone is simpler.
* [Azure App Service VNet Integration](https://docs.microsoft.com/en-us/azure/app-service/web-sites-integrate-with-vnet) - allows App Service to access resources in VNet rather than restricting traffic coming into App Service.
* [Azure App Service Environment](https://docs.microsoft.com/en-us/azure/app-service/environment/intro) - places App Service in a VNet. This is a dedicated App Service SKU rather than multi-tenant.

### TODO
* Build the ARM template.
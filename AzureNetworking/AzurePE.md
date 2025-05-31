{% include nov.md %}

# Azure Private Endpoint

Azure Private Endpoint is a network interface that connects you privately and securely to a service powered by Azure Private Link. Private Endpoint uses a private IP address from your virtual network, effectively bringing the service into your virtual network. This allows you to access Azure services (like Azure Storage, Azure Cosmos DB, Azure SQL Database, etc.) and Azure hosted customer/partner services over your private network, eliminating exposure to the public internet.

## Why use Azure Private Endpoint?

* **Enhanced Security:** Data is transmitted exclusively over the Azure backbone network, never traversing the public internet. This significantly reduces the risk of data exfiltration and other security threats.
* **Simplified Network Architecture:** Eliminates the need for complex firewall rules, NAT devices, or bastion hosts to secure connectivity to Azure services.
* **Private IP Address Access:** Services become accessible using private IP addresses within your virtual network, just like any other resource in your VNet.
* **Data Exfiltration Protection:** Helps prevent data exfiltration by ensuring that data only flows through authorized private connections.
* **On-premises Connectivity:** Extends private connectivity from your on-premises network to Azure services when combined with VPN or ExpressRoute.

## Resources that can be mapped with Azure Private Endpoint

Azure Private Endpoint supports a wide range of Azure services. Here's a list of commonly used resources that can be mapped with a Private Endpoint:

### Storage

* **Azure Storage Accounts:**
    * Blob (including Data Lake Storage Gen2)
    * Queue
    * Table
    * File
* **Azure Data Lake Storage Gen2 (via Storage Accounts)**

### Databases

* **Azure SQL Database**
* **Azure Database for MySQL**
* **Azure Database for PostgreSQL**
* **Azure Database for MariaDB**
* **Azure Cosmos DB:**
    * SQL (Core) API
    * MongoDB API
    * Cassandra API
    * Gremlin API
    * Table API
* **Azure Synapse Analytics (Dedicated SQL Pool)**
* **Azure Cache for Redis**

### Analytics & IoT

* **Azure Key Vault**
* **Azure Event Hubs**
* **Azure Service Bus**
* **Azure IoT Hub**
* **Azure Data Factory**
* **Azure Stream Analytics (Input/Output to private linked resources)**
* **Azure HDInsight (for specific components like Ranger)**
* **Azure Databricks (Control Plane and Data Plane)**

### Identity & Security

* **Azure Key Vault**
* **Azure Container Registry**

### Web & Networking

* **Azure App Service:**
    * Web Apps (Standard, Premium, Isolated plans)
    * Function Apps
* **Azure API Management (Developer, Basic, Standard, Premium tiers)**
* **Azure Application Gateway (V2 SKU)**
* **Azure Front Door Standard/Premium (Origin via Private Link)**
* **Azure Monitor (Log Analytics workspaces and Application Insights resources)**
* **Azure Backup (Recovery Services Vault)**

### Other

* **Azure Machine Learning Workspace**
* **Azure Cognitive Services**
* **Azure Search**
* **Azure Kubernetes Service (AKS) - for API server access**

---

**Note:** This list is not exhaustive and Azure is continuously adding support for more services with Private Link. Always refer to the [official Azure Private Link documentation](https://learn.microsoft.com/en-us/azure/private-link/private-link-overview#availability) for the most up-to-date information on supported services and regional availability.

## How to Configure

For detailed steps on how to configure Azure Private Endpoint for specific services, please refer to the official Azure documentation for each service. Generally, the process involves:

1.  Creating a Virtual Network (VNet) and a subnet.
2.  Creating the Azure service resource (e.g., Storage Account, SQL Database).
3.  Creating a Private Endpoint for the service within your VNet.
4.  Configuring DNS resolution (either using Azure Private DNS Zone or your own DNS servers) to resolve the service's FQDN to the Private Endpoint's private IP address.

## Further Reading

* [What is Azure Private Link?](https://learn.microsoft.com/en-us/azure/private-link/private-link-overview)
* [What is Azure Private Endpoint?](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview)
* [Azure Private Link pricing](https://azure.microsoft.com/en-us/pricing/details/private-link/)

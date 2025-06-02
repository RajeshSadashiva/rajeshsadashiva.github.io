{% include nov.md %}

# Migrating Azure Basic Load Balancer to Standard

This guide outlines the steps to migrate an existing Azure Basic Load Balancer to a Standard Load Balancer. Standard Load Balancers offer enhanced features, improved performance, and higher availability compared to Basic Load Balancers.

**Important Considerations Before Migration:**

* **Downtime:** This process involves reconfiguring network components and will likely result in some downtime for your applications. Plan for a maintenance window.
* **IP Addresses:** Standard Load Balancers require Standard SKU Public IP addresses. If your current Basic Load Balancer uses Basic Public IPs, you'll need to create new Standard Public IPs.
* **Associated Resources:** Ensure all backend pool members (VMs, VM Scale Sets) are in the same virtual network as the new Standard Load Balancer.
* **Outbound Connectivity:** Standard Load Balancers are secure by default. If your backend VMs require outbound internet access, you'll need to configure explicit outbound rules or use a NAT Gateway. Basic Load Balancers provide automatic outbound SNAT.
* **Health Probes:** Review and reconfigure health probes for the Standard Load Balancer.
* **Pricing:** Standard Load Balancers have a different pricing model than Basic Load Balancers.

## Migration Steps:

The migration process generally involves creating a new Standard Load Balancer and then re-associating your backend resources. There is no in-place upgrade path from Basic to Standard.

### Method 1: Manual Migration (Recommended for Control)

1.  **Identify Current Configuration:**
    * Note down all frontend IP configurations, backend pools, health probes, and load balancing rules of your existing Basic Load Balancer.
    * Identify the virtual network and subnets where your backend resources reside.

2.  **Create New Standard Public IP Addresses (if needed):**
    * If your Basic Load Balancer uses Basic Public IPs, create new **Standard SKU Public IP addresses** in the same region as your resources.
        ```azurecli
        az network public-ip create \
          --resource-group <resource-group-name> \
          --name <standard-public-ip-name> \
          --sku Standard \
          --allocation-method Static \
          --location <location>
        ```

3.  **Create a New Standard Load Balancer:**
    * Create a new Standard SKU Load Balancer in the same virtual network.
        ```azurecli
        az network lb create \
          --resource-group <resource-group-name> \
          --name <standard-lb-name> \
          --sku Standard \
          --location <location>
        ```

4.  **Configure Frontend IP Configurations:**
    * Add frontend IP configurations to the new Standard Load Balancer, associating them with the new Standard Public IP addresses you created (or existing private IPs if internal LB).
        ```azurecli
        az network lb frontend-ip create \
          --resource-group <resource-group-name> \
          --lb-name <standard-lb-name> \
          --name <frontend-ip-name> \
          --public-ip-address <standard-public-ip-name>
        ```

5.  **Configure Backend Pools:**
    * Create new backend pools on the Standard Load Balancer.
        ```azurecli
        az network lb address-pool create \
          --resource-group <resource-group-name> \
          --lb-name <standard-lb-name> \
          --name <backend-pool-name>
        ```

6.  **Configure Health Probes:**
    * Create health probes for the Standard Load Balancer that match your application's requirements.
        ```azurecli
        az network lb probe create \
          --resource-group <resource-group-name> \
          --lb-name <standard-lb-name> \
          --name <probe-name> \
          --protocol Tcp \
          --port 80 \
          --interval 5 \
          --threshold 2
        ```

7.  **Configure Load Balancing Rules:**
    * Create load balancing rules that direct traffic from the frontend to the backend pools using the defined health probes.
        ```azurecli
        az network lb rule create \
          --resource-group <resource-group-name> \
          --lb-name <standard-lb-name> \
          --name <rule-name> \
          --protocol Tcp \
          --frontend-port 80 \
          --backend-port 80 \
          --frontend-ip-name <frontend-ip-name> \
          --backend-pool-name <backend-pool-name> \
          --probe-name <probe-name>
        ```

8.  **Detach Backend VMs from Basic Load Balancer:**
    * Remove network interfaces of backend VMs from the backend pool of the Basic Load Balancer. This will cause downtime.
        ```azurecli
        # Example for a single VM NIC
        az network nic ip-config update \
          --resource-group <resource-group-name> \
          --nic-name <nic-name> \
          --name <ip-config-name> \
          --lb-address-pool "" # Remove association
        ```

9.  **Attach Backend VMs to Standard Load Balancer:**
    * Add the network interfaces of your backend VMs to the backend pool of the new Standard Load Balancer.
        ```azurecli
        # Example for a single VM NIC
        az network nic ip-config update \
          --resource-group <resource-group-name> \
          --nic-name <nic-name> \
          --name <ip-config-name> \
          --lb-address-pool <backend-pool-name-standard-lb>
        ```
    * For VM Scale Sets, you'll update the scale set configuration to use the new Standard Load Balancer.

10. **Test and Verify:**
    * Thoroughly test your application's connectivity and functionality through the new Standard Load Balancer.
    * Monitor logs and metrics to ensure everything is working as expected.

11. **Delete Basic Load Balancer (After Successful Migration):**
    * Once you've confirmed that your application is fully functional with the Standard Load Balancer, delete the old Basic Load Balancer and its associated Basic Public IP addresses (if any).
        ```azurecli
        az network lb delete \
          --resource-group <resource-group-name> \
          --name <basic-lb-name>
        ```

### Method 2: Using the Azure Portal (Visual Approach)

1.  **Document Basic Load Balancer Configuration:** Go to your Basic Load Balancer in the Azure portal and note down all configurations (frontend IPs, backend pools, health probes, rules).
2.  **Create New Standard Public IP Addresses:** If needed, create new Standard SKU Public IP addresses.
3.  **Create New Standard Load Balancer:**
    * Navigate to "Load balancers" in the Azure portal.
    * Click "Create."
    * Select "Standard" for the SKU.
    * Configure frontend IP, backend pools, health probes, and load balancing rules as per your documented Basic Load Balancer settings, using the new Standard Public IPs.
4.  **Update Backend VM NICs:**
    * For each backend VM, go to its "Networking" settings.
    * Click on the network interface.
    * Under "IP configurations," select the IP configuration.
    * Change the associated Load Balancer from the Basic one to the new Standard Load Balancer.
    * For VM Scale Sets, update the Load Balancer configuration within the VM Scale Set settings.
5.  **Test and Verify:** Test your application thoroughly.
6.  **Delete Basic Load Balancer:** Once confident, delete the Basic Load Balancer from the Azure portal.

### Troubleshooting Tips:

* **Outbound Connectivity Issues:** If your backend VMs lose internet access, ensure you have configured outbound rules for your Standard Load Balancer or deployed a NAT Gateway.
* **Health Probe Failures:** Verify that your application is listening on the configured port and that network security groups (NSGs) allow traffic from the Load Balancer to the backend VMs on the probe port.
* **NSG Rules:** Standard Load Balancers require explicit NSG rules to allow inbound traffic. Ensure your NSGs permit traffic from the Load Balancer's frontend IP to your backend VMs.
* **IP Address SKU Mismatch:** Double-check that all associated Public IP addresses are Standard SKU.

## Conclusion

Migrating from an Azure Basic Load Balancer to a Standard Load Balancer is a manual process but provides significant benefits in terms of features and reliability. Careful planning and thorough testing are crucial for a smooth transition and to minimize application downtime.

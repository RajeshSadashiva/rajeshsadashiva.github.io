
Azure VNet to On-Premise VPN Gateway Connection
This repository provides guidance and potentially configuration files for establishing a secure IPsec VPN tunnel between an Azure Virtual Network (VNet) and an on-premises network. This connection allows resources in your Azure VNet to communicate directly with resources in your on-premises data center, extending your corporate network into Azure.
Table of Contents
 * Introduction
 * Prerequisites
 * Architecture
 * Azure Configuration Steps
   * 1. Create a Virtual Network (VNet)
   * 2. Create a Gateway Subnet
   * 3. Create a Virtual Network Gateway
   * 4. Create a Local Network Gateway
   * 5. Create the Connection
 * On-Premise Configuration Steps
 * Verification
 * Troubleshooting
 * Security Considerations
 * Contributing
 * License
Introduction
This setup utilizes an Azure VPN Gateway, which is a type of virtual network gateway that sends encrypted traffic across a public connection to your on-premises network. The most common scenario for connecting to on-premises networks is a Site-to-Site (S2S) VPN connection using IPsec/IKE (IKEv1 or IKEv2).
Prerequisites
Before you begin, ensure you have the following:
 * Azure Subscription: An active Azure subscription.
 * Azure Permissions: Permissions to create and manage Azure networking resources (VNets, VPN Gateways, Local Network Gateways).
 * On-Premise VPN Device: A compatible on-premises VPN device (e.g., Cisco ASA, FortiGate, Palo Alto, Windows Server RRAS, etc.) with a public static IP address. Refer to the Azure VPN Gateway documentation for compatible devices.
 * On-Premise Network Information:
   * Your on-premises public static IP address for the VPN device.
   * The IP address ranges/subnets of your on-premises network that need to communicate with Azure (e.g., 192.168.1.0/24).
 * Azure VNet Information:
   * Desired Azure VNet name and address space (e.g., 10.0.0.0/16).
   * Desired subnet within the VNet (e.g., 10.0.1.0/24).
   * Desired GatewaySubnet address space (e.g., /27 or larger, like 10.0.255.0/27).
 * Shared Key: A pre-shared key (PSK) to establish the VPN tunnel. This key must be identical on both the Azure VPN Gateway and your on-premises VPN device.
Architecture
+-------------------+                      +-------------------------+
| On-Premise Network|                      |   Azure Cloud Network   |
|                   |                      |                         |
| +---------------+ |      Internet      | +---------------------+ |
| |  On-Premise   | | <----------------> | |  Azure Virtual      | |
| |  VPN Device   | |  (IPsec VPN Tunnel)| |  Network Gateway    | |
| | (Public IP:   | |                    | | (Public IP:         | |
| |  X.X.X.X)     | |                    | |  Y.Y.Y.Y)           | |
| +---------------+ |                      | +---------------------+ |
|         |         |                      |           |             |
|         |         |                      |           |             |
| +-----------------+                      | +---------------------+ |
| |  Internal       |                      | |  Azure VNet         | |
| |  Network Subnets|                      | |  (e.g., 10.0.0.0/16)| |
| |  (e.g., 192.168.|                      | |   - Subnet A        | |
| |  1.0/24)        |                      | |   - Subnet B        | |
| +-----------------+                      | +---------------------+ |
+-------------------+                      +-------------------------+

Azure Configuration Steps
These steps can be performed using the Azure Portal, Azure CLI, or Azure PowerShell. For simplicity, we'll outline the conceptual steps. Refer to the official Azure documentation for detailed commands.
1. Create a Virtual Network (VNet)
If you don't already have one, create an Azure VNet where your resources will reside.
 * Name: myVNet (example)
 * Address Space: 10.0.0.0/16 (example)
 * Region: Your desired Azure region.
2. Create a Gateway Subnet
A dedicated subnet named GatewaySubnet is required within your VNet for the VPN Gateway. This subnet must be named GatewaySubnet.
 * VNet: myVNet
 * Subnet Name: GatewaySubnet
 * Address Range: At least /27 (e.g., 10.0.255.0/27). A /27 provides 32 addresses, which is sufficient for most scenarios. For future scalability, consider a larger subnet like /26 or /25.
3. Create a Virtual Network Gateway
This is the Azure-side endpoint of your VPN connection.
 * Name: myVNetGateway (example)
 * Region: Must be the same as your VNet.
 * Gateway type: VPN
 * VPN type: Route-based (recommended for S2S connections)
 * SKU: Choose a suitable SKU based on your performance and redundancy requirements (e.g., VpnGw1, VpnGw2, VpnGw3, VpnGw4, VpnGw5, Basic). Basic SKU has limitations.
 * Virtual network: Select myVNet.
 * Gateway subnet: Will automatically select GatewaySubnet.
 * Public IP address: Create a new public IP address for the gateway.
   * Name: myVNetGatewayPIP (example)
   * SKU: Standard (recommended for production)
   * Assignment: Static
4. Create a Local Network Gateway
The Local Network Gateway represents your on-premises VPN device and its associated network.
 * Name: myOnPremiseGateway (example)
 * IP address: Enter the public static IP address of your on-premises VPN device (e.g., X.X.X.X).
 * Address spaces: Add the IP address ranges/subnets of your on-premises network that you want to reach from Azure (e.g., 192.168.1.0/24, 192.168.2.0/24).
5. Create the Connection
This step links your Virtual Network Gateway to your Local Network Gateway.
 * Name: myVNetToOnPremConnection (example)
 * Connection type: Site-to-site (IPsec)
 * Virtual network gateway: Select myVNetGateway.
 * Local network gateway: Select myOnPremiseGateway.
 * Shared key (PSK): Enter a strong pre-shared key. This key must exactly match the key you configure on your on-premises VPN device.
 * IKE Protocol: Select IKEv2 (recommended for modern devices) or IKEv1 if your on-premises device only supports it.
On-Premise Configuration Steps
This is the most critical and varied part. The exact steps depend entirely on your on-premises VPN device's make and model (e.g., Cisco ASA, FortiGate, pfSense, Windows Server RRAS, etc.).
General steps for on-premises VPN device configuration:
 * Identify Public IP: Ensure your on-premises VPN device has a public static IP address.
 * IPsec Tunnel Configuration:
   * Tunnel Type: IPsec Site-to-Site VPN.
   * Local Network (On-Premise): Define your local internal network subnets (e.g., 192.168.1.0/24).
   * Remote Network (Azure): Define your Azure VNet's address space (e.g., 10.0.0.0/16).
   * Remote Gateway (Azure): Enter the public IP address of your Azure Virtual Network Gateway (e.g., Y.Y.Y.Y).
   * Pre-Shared Key (PSK): Enter the exact same shared key you configured on the Azure connection.
   * IKE/IPsec Parameters:
     * IKE Phase 1 (Main Mode/Aggressive Mode):
       * Encryption: AES256 (recommended)
       * Hashing: SHA256 (recommended)
       * Diffie-Hellman Group: Group 2, 14, 24 (ensure compatibility with Azure's supported groups)
       * Lifetime: 28800 seconds (8 hours)
     * IKE Phase 2 (Quick Mode):
       * Encryption: AES256 (recommended)
       * Hashing: SHA256 (recommended)
       * Perfect Forward Secrecy (PFS) Group: Same as Phase 1 or lower (e.g., Group 2, 14, 24).
       * Lifetime: 3600 seconds (1 hour)
     * NAT Traversal (NAT-T): Enable if your on-premises device is behind a NAT.
     * Dead Peer Detection (DPD): Enable.
   * Firewall Rules: Ensure your on-premises firewall allows UDP ports 500 (IKE) and 4500 (IPsec NAT-T) to and from the Azure VPN Gateway's public IP address.
   * Routing: Configure static routes or routing policies on your on-premises device to direct traffic destined for the Azure VNet address space (10.0.0.0/16) through the VPN tunnel interface.
Refer to your VPN device's documentation for specific configuration commands and GUI steps.
Verification
After configuring both ends, verify the connection:
 * Azure Portal:
   * Navigate to your Virtual Network Gateway -> "Connections".
   * The connection status should eventually show "Connected".
   * Check "Metrics" for data ingress/egress.
 * On-Premise VPN Device:
   * Check the VPN tunnel status on your device. It should indicate the tunnel is up and active.
   * Look for IPsec SA (Security Association) entries.
 * Ping/Traceroute:
   * From a VM in your Azure VNet, try to ping an on-premises server.
   * From an on-premises server, try to ping an Azure VM's private IP address.
   * Ensure firewall rules on both the Azure VMs (Network Security Groups) and on-premises servers allow ICMP traffic.
 * Network Tools: Use ping, traceroute, telnet, or netcat to test connectivity to specific ports/services.
Troubleshooting
 * Shared Key Mismatch: The most common issue. Double-check that the pre-shared key is identical on both ends.
 * IP Address Mismatch: Ensure the public IP of the on-premises device configured in Azure's Local Network Gateway matches the actual public IP.
 * Address Space Mismatch: Verify that the on-premises address spaces configured in Azure's Local Network Gateway precisely match the internal networks you want to expose. Similarly, ensure the Azure VNet address space is correctly defined on the on-premises device.
 * Firewall Issues: Check both Azure Network Security Groups (NSGs) and your on-premises firewall rules. Ensure UDP 500 and 4500 are open.
 * IKE/IPsec Parameter Mismatch: Ensure encryption algorithms, hashing algorithms, Diffie-Hellman groups, and lifetimes for both IKE Phase 1 and Phase 2 match between Azure and your on-premises device. Azure has default policies, and custom policies can be used if needed.
 * Routing Issues: Verify that routes are correctly configured on both ends to direct traffic through the VPN tunnel.
 * NAT-T Issues: If your on-premises device is behind a NAT, ensure NAT-T is enabled on both sides.
 * Logs: Review logs on both the Azure VPN Gateway (via Azure Monitor) and your on-premises VPN device for error messages.
 * Azure VPN Gateway Health: Check the health status of your Azure VPN Gateway in the Azure portal.
 * Public IP Issues: Ensure the public IP addresses are truly static and publicly routable.
Security Considerations
 * Strong Shared Key: Use a long, complex, and random pre-shared key.
 * Least Privilege: Limit the network access between on-premises and Azure to only what's necessary using NSGs and on-premises firewall rules.
 * Monitoring: Implement robust monitoring for your VPN connection to detect outages or unusual traffic patterns.
 * Regular Updates: Keep your on-premises VPN device firmware and Azure resources up to date.
 * Custom IPsec Policies: For enhanced security, consider using custom IPsec/IKE policies if your on-premises device supports stronger algorithms than Azure's default policies.
Contributing
If you have improvements to this guide, specific configuration examples for different on-premises VPN devices, or troubleshooting tips, please feel free to open a pull request or issue.
License
This project is licensed under the MIT License - see the LICENSE file for details.

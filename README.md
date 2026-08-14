# Morpheus BIG-IP Plugin

The Morpheus BIG-IP Plugin integrates Morpheus with F5 BIG-IP load balancers via the iControl REST API, enabling full lifecycle management of virtual servers, pools, nodes, health monitors, profiles, policies, iRules, persistence profiles, and SSL certificates from within the Morpheus UI.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Repository structure](#repository-structure)
- [Building the plugin](#building-the-plugin)
- [License](#license)
- [Installing](#installing)
- [Detailed Usage Steps](#detailed-usage-steps)
- [API Endpoints](#api-endpoints)

---

## Features

### Load Balancer Management

Full lifecycle management of F5 BIG-IP load balancers as Morpheus network load balancer integrations. Supports:

- Virtual server (VIP) creation, update, and deletion
- Pool creation and management, including pool member add/remove
- Node creation and management
- Health monitor creation and management (per monitor type)
- Profile assignment (client-SSL, server-SSL, HTTP, TCP, etc.)
- LTM policy and policy rule management
- iRule management
- Persistence profile management
- SSL certificate management
- Partition management
- Manual VIP address entry or VIP address assignment from a Morpheus network pool
- Instance-to-load-balancer association

### Cloud Sync

Morpheus synchronises the following BIG-IP resources for inventory:

- Partitions
- Virtual servers (instances)
- Pools and pool members
- Nodes
- Health monitors
- LTM profiles
- LTM policies
- Persistence profiles
- SSL certificates
- iRules

---

## Requirements

| Requirement | Version |
|-------------|---------|
| Morpheus | 8.1.0 or later |
| Java | 11 or later |
| Gradle | Use the included Gradle wrapper (`./gradlew`) |

Additional prerequisites:

- A running F5 BIG-IP device with iControl REST enabled (accessible via HTTPS)
- A BIG-IP user account with the **Administrator** role (or equivalent) on the target partition; the API requires full read/write access to LTM objects
- Network access from the Morpheus appliance to the BIG-IP management IP on the configured API port (default: 443) over HTTPS
- Valid TLS configuration on the BIG-IP management interface; Morpheus accepts the presented certificate (self-signed certificates are supported via the plugin's SSL settings)

---

## Repository structure

```
src/main/groovy/com/morpheusdata/bigip/
├── BigIpPlugin.groovy                - Plugin entry point; registers BigIpProvider and BigIpOptionSourceProvider
├── BigIpProvider.groovy              - LoadBalancerProvider implementation; all CRUD operations and sync orchestration
├── BigIpOptionSourceProvider.groovy  - UI option source data (partitions, monitors, profiles, etc.)
├── sync/
│   ├── CertificateSync.groovy        - Syncs SSL certificates
│   ├── HealthMonitorSync.groovy      - Syncs health monitors
│   ├── IRuleSync.groovy              - Syncs iRules
│   ├── InstanceSync.groovy           - Syncs virtual server instances
│   ├── NodesSync.groovy              - Syncs nodes
│   ├── PartitionSync.groovy          - Syncs partitions
│   ├── PersistenceSync.groovy        - Syncs persistence profiles
│   ├── PolicySync.groovy             - Syncs LTM policies
│   ├── PoolSync.groovy               - Syncs pools and pool members
│   └── ProfileSync.groovy            - Syncs LTM profiles
├── controllers/                      - Custom UI controllers (if any)
└── util/
    └── BigIpUtility.groovy           - Shared helpers (partitioned name builder, external ID conversion, etc.)
src/assets/                           - Plugin icons (bigip.svg, bigip-dark-mode.svg)
src/test/groovy/                       - Spock unit tests
build.gradle, gradle.properties        - Build configuration and plugin metadata
```

---

## Building the plugin

Run the following command to compile and package the plugin jar:

```bash
./gradlew clean build
```

The packaged jar will be written to `build/libs/`.

To execute tests, use the following command:

```bash
./gradlew test
```

---

## License

This project is licensed under the Apache License 2.0.

See the [LICENSE](LICENSE) file for details.

---

## Installing

1. Build the plugin (see [Building the plugin](#building-the-plugin)) or download a released jar.
2. In Morpheus, navigate to **Administration > Integrations > Plugins**.
3. Click **Add** and upload the `morpheus-bigip-loadbalancer-plugin-<version>.jar` from `build/libs/`.
4. Navigate to **Infrastructure > Load Balancers > Add** and select **F5 BigIp** to configure the integration.

---

## Detailed Usage Steps

### Adding a BIG-IP Load Balancer Integration

1. Go to **Infrastructure > Load Balancers > Add**.
2. Select **F5 BigIp** as the load balancer type.
3. Enter the **API Host** (BIG-IP management IP or hostname) and **API Port** (default: 443).
4. Provide credentials: select **Username / Password** (or a stored credential) and enter the **Username** and **Password**.
5. Optionally enter the **Management URL** for direct BIG-IP UI links.
6. Configure **Allow Manual VIP Entry** and optionally select a **Network Pool** for automated VIP address allocation.
7. Save. Morpheus validates the connection and begins syncing BIG-IP inventory.

### Running or Verifying Inventory Synchronisation

1. After saving the integration, Morpheus immediately runs a full sync.
2. To verify, go to the load balancer detail page and review the **Virtual Servers**, **Pools**, **Nodes**, **Monitors**, **Profiles**, **Policies**, **iRules**, **Persistence**, and **Certificates** tabs.
3. To trigger a manual refresh, use **Actions > Refresh** on the load balancer.

### Creating a Virtual Server

1. From the load balancer detail page, go to the **Virtual Servers** tab and click **Add**.
2. Enter the **Virtual Service Name**, **VIP Address** (manual or assigned from a network pool), and **Port**.
3. Assign a **Pool**, **Profiles**, **Persistence Profile**, **Policies**, **iRules**, and **SSL Certificate** as required.
4. Save. The plugin creates the virtual server on BIG-IP via the iControl REST API.

### Creating a Pool and Adding Pool Members

1. From the **Pools** tab, click **Add**.
2. Enter the pool name and select a **Health Monitor**.
3. Save the pool, then add **Members** by specifying node name and port.

### Creating Nodes

1. From the **Nodes** tab, click **Add**.
2. Provide the node name and IP address.
3. Save. The node is created in BIG-IP and made available for pool membership.

### Creating Health Monitors

1. From the **Monitors** tab, click **Add**.
2. Select the monitor type (HTTP, HTTPS, TCP, etc.) and configure interval, timeout, and send/receive strings as applicable.
3. Save.

### Associating an Instance with the Load Balancer

1. From **Provisioning > Instances**, open an instance.
2. Click **Actions > Load Balancers**.
3. Select the BIG-IP integration, virtual server, and pool membership settings.
4. Save. Morpheus adds the instance as a node and pool member on BIG-IP.

### Removing Managed BIG-IP Resources

1. Deleting a virtual server, pool, or node from the Morpheus UI removes it from BIG-IP via the API.
2. Removing a load balancer association from an instance removes the corresponding pool member and, if no longer referenced, the node.

---

## API Endpoints

This plugin communicates with the **F5 BIG-IP iControl REST API** at `https://<bigip-host>:<api-port>/mgmt`. All calls use HTTPS.

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/mgmt/tm/auth/partition` | GET | List partitions / validate credentials |
| `/mgmt/tm/ltm/virtual` | GET | List virtual servers |
| `/mgmt/tm/ltm/virtual/{name}` | GET / PUT / DELETE | Get, update, or delete a virtual server |
| `/mgmt/tm/ltm/virtual/{name}` | POST | Create a virtual server |
| `/mgmt/tm/ltm/pool` | GET | List pools |
| `/mgmt/tm/ltm/pool/{name}` | GET / PUT / DELETE | Get, update, or delete a pool |
| `/mgmt/tm/ltm/pool/{name}/members` | GET / POST / DELETE | List, add, or remove pool members |
| `/mgmt/tm/ltm/node` | GET | List nodes |
| `/mgmt/tm/ltm/node/{name}` | GET / PUT / DELETE | Get, update, or delete a node |
| `/mgmt/tm/ltm/monitor/{type}` | GET | List monitors by type |
| `/mgmt/tm/ltm/monitor/{type}/{name}` | GET / PUT / DELETE | Get, update, or delete a monitor |
| `/mgmt/tm/ltm/policy` | GET | List LTM policies |
| `/mgmt/tm/ltm/policy/{name}/rules/{rule}` | GET / PUT | Get or update a policy rule |
| `/mgmt/tm/ltm/profile` | GET | List profiles (all types) |
| `/mgmt/tm/ltm/profile/client-ssl/{name}` | GET | Get client-SSL profile |
| `/mgmt/tm/ltm/persistence` | GET | List persistence profiles |
| `/mgmt/tm/ltm/rule` | GET | List iRules |
| `/mgmt/tm/sys/file/ssl-cert` | GET | List SSL certificates |

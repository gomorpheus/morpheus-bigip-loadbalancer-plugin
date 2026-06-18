# Morpheus BigIP Plugin

This plugin provides load balancer integration between [F5 BIG-IP](https://www.f5.com/products/big-ip-services) and [Morpheus](https://morpheusdata.com). It enables BIG-IP inventory sync, virtual server management, pool and node management, policy and rule management, SSL profile management, and health monitor management from within the Morpheus platform.

## Requirements

| Component | Minimum Version |
|-----------|----------------|
| Morpheus | 8.1.0 |

## Installation

1. Download the latest `.jar` from the [Releases](https://github.com/HewlettPackard/morpheus-bigip-loadbalancer-plugin/releases) page, or [build it yourself](#building).
2. In Morpheus, navigate to **Administration → Integrations → Plugins**.
3. Click **Browse** and upload the `.jar` file.
4. The **BigIp** load balancer type will appear after the plugin loads.

## Configuration

When adding a BIG-IP load balancer in Morpheus (**Infrastructure → Load Balancers → Add Load Balancer**), provide the following:

| Field | Description |
|-------|-------------|
| **Api Host** | BIG-IP API hostname or IP address |
| **Api Port** | BIG-IP API port; defaults to `443` |
| **Credentials** | Morpheus credential containing the BIG-IP username and password |
| **Username** | BIG-IP username when using local credentials |
| **Password** | BIG-IP password when using local credentials |
| **Management URL** | Optional BIG-IP management UI URL |
| **Allow Vip Entry** | Allows manual VIP entry for virtual server configuration |
| **Vip Pools** | Optional Morpheus network pools used for VIP assignment |
| **Virtual Name** | Optional naming pattern for virtual servers |
| **Pool Name** | Optional naming pattern for pools |
| **Server Name** | Optional naming pattern for nodes or servers |

Credentials can also be stored as a Morpheus [Credential](https://docs.morpheusdata.com/en/latest/administration/credentials/credentials.html) and selected at load balancer setup time.

## Features

### Load Balancer Sync
The following BIG-IP resources are discovered and kept in sync:

- **Partitions** — BIG-IP administrative partitions
- **Nodes** — backend nodes used by load balancing pools
- **Health Monitors** — monitor definitions assigned to pools
- **Pools** — load balancing pools and members
- **Policies and Rules** — traffic policies and matching rules
- **Profiles** — HTTP and SSL profile definitions
- **Certificates** — BIG-IP certificates available for SSL profiles
- **Persistence Profiles** — persistence settings available to virtual servers
- **iRules** — BIG-IP iRules available to virtual servers
- **Virtual Servers** — BIG-IP virtual server instances

### Virtual Servers
Virtual servers can be created and managed from Morpheus. Supported operations include:

- Create, update, and delete virtual servers
- Assign pools, profiles, policies, persistence profiles, and iRules
- Use manual VIP values or VIP addresses from Morpheus network pools
- Update virtual server membership from Morpheus instance changes

### Pools, Nodes, and Monitors
The plugin manages core load balancing components used by virtual servers. Supported operations include:

- Create, update, validate, and delete pools
- Create, update, validate, and delete nodes
- Create, update, validate, and delete health monitors
- Assign pool members and health monitors from synced BIG-IP inventory

### Policies, Rules, and Profiles
The plugin exposes BIG-IP traffic policy and profile configuration in Morpheus. Supported operations include:

- Create, validate, and delete policies
- Create, validate, and delete policy rules
- Create, update, and delete SSL and HTTP profiles
- Select controls, requirements, strategies, rule fields, operators, and target pools from BIG-IP option sources

## Building

```bash
./gradlew shadowJar
```

The plugin JAR will be written to `build/libs/`.

## License

Copyright 2022 Morpheus Data, LLC. Licensed under the [Apache License, Version 2.0](LICENSE).

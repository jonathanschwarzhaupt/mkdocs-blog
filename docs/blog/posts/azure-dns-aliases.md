---
date:
  created: 2025-02-03
tags:
    - Azure
categories:
  - Cloud
links:
  - External links:
    - Docs: https://learn.microsoft.com/en-us/azure/dns/dns-alias
---

# Azure DNS aliases can reference other Azure resources

Azure DNS is Microsoft's hosting service for DNS domains. It allows users to manage their infrastructure and related related domain information in one central place.

<!-- more -->

This alone could be a decent argument for Azure DNS. But one of the key benefits is the tight integration with Azure's resources:

Azure DNS aliases provide dynamic references to Azure resources and can be created at the zone apex level.
This provides key benefits such as automatic updates of the DNS record set when an underlying IP address changes thus preventing dangling DNS records, load balancing of the apex domain, and direct reference to Azure Content Delivery Network endpoints.

They key here is the tight integration of Azure DNS with the overall Azure platform. It is able to reference the Azure resource instead of its IP address.

---
date:
  created: 2025-01-28
tags:
    - Azure
    - Blob Storage
categories:
  - Cloud
---

# Short rundown of Azure blob storage's access tiers

There are different access tiers for blobs residing in Azure Storage. <!-- more --> These tiers include
- Hot
- Cool
- Cold
- Archive
The storage tier is set on the storage account level. Options differ with respect to storage and access cost, minimum storage duration, and latency (time until data is retrieved) and are thus suited to different scenarios.

## Hot
This tier is optimized for frequent read or writes. It is thus suited for actively used data. It incurs the highest storage cost, but lowest access cost and is the default option when creating storage accounts.

## Cool
This tier is suited for storing large amounts of infrequently accessed data. Data should remain in this tier for at least 30 days, but can be retrieved instantaneously. It thus represents a short-term backup and disaster recovery option. Compared to the hot tier, storage cost is lower but access cost higher

## Cold
In the cold tier, data should remain in the tier for at least 90 days. Storage cost is even lower, but access cost higher compared to the cool tier.

## Archive
Here, long-term retention of data with at least 180 days and latency of several hours before data becomes available are the key characteristics. Use cases often include secondary backups, original raw data and legally required compliance information. Logically, it offers the lowest storage cost, but highest access costs.

By selecting the right access tier, you can store your data in the most cost-effective manner.

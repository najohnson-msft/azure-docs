---
title: Supported versions - Azure Database for PostgreSQL - Flexible Server
description: Describes the supported PostgreSQL major and minor versions in Azure Database for PostgreSQL - Flexible Server.
ms.author: sunila
author: sunilagarwal
ms.service: postgresql
ms.subservice: flexible-server
ms.topic: conceptual
ms.date: 04/14/2022
---

# Supported PostgreSQL major versions in Azure Database for PostgreSQL - Flexible Server



Azure Database for PostgreSQL - Flexible Server currently supports the following major versions:

## PostgreSQL version 13

The current minor release is **13.6**. Refer to the [PostgreSQL documentation](https://www.postgresql.org/docs/13/static/release-13-6.html) to learn more about improvements and fixes in this release. New servers will be created with this minor version. 

## PostgreSQL version 12

The current minor release is **12.10**. Refer to the [PostgreSQL documentation](https://www.postgresql.org/docs/12/static/release-12-10.html) to learn more about improvements and fixes in this release. New servers will be created with this minor version. Your existing servers will be automatically upgraded to the latest supported minor version in your future scheduled maintenance window.

## PostgreSQL version 11

The current minor release is **11.15**. Refer to the [PostgreSQL documentation](https://www.postgresql.org/docs/11/static/release-11-15.html) to learn more about improvements and fixes in this release. New servers will be created with this minor version. Your existing servers will be automatically upgraded to the latest supported minor version in your future scheduled maintenance window.

## PostgreSQL version 10 and older

We do not support PostgreSQL version 10 and older for Azure Database for PostgreSQL - Flexible Server. Please use the [Single Server](../concepts-supported-versions.md) deployment option if you require older versions.

## Managing upgrades

The PostgreSQL project regularly issues minor releases to fix reported bugs. Azure Database for PostgreSQL automatically patches servers with minor releases during the service's monthly deployments.

Automation for major version upgrade is not yet supported. For example, there is currently no automatic upgrade from PostgreSQL 11 to PostgreSQL 12.<!-- To upgrade to the next major version, create a [database dump and restore](howto-migrate-using-dump-and-restore.md) to a server that was created with the new engine version.-->

<!--
## Next steps

For information on supported PostgreSQL extensions, see [the extensions document](concepts-extensions.md).
-->
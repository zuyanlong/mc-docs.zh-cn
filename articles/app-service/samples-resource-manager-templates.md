---
title: Azure 资源管理器模板示例
description: 查找某些常用应用服务方案的 Azure 资源管理器模板示例。 了解如何自动完成应用服务部署或管理任务。
author: tfitzmac
tags: azure-service-management
ms.topic: sample
origin.date: 08/26/2020
ms.date: 10/19/2020
ms.author: v-tawe
ms.custom: mvc, fasttrack-edit
ms.openlocfilehash: 5821091e2bd45ca3e714b21e90d49630c79f22d2
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170487"
---
# <a name="azure-resource-manager-templates-for-app-service"></a>应用服务的 Azure 资源管理器模板

下表包含 Azure 应用服务的 Azure 资源管理器模板链接。 有关如何在创建应用模板时避免常见错误的建议，请参阅[有关使用 Azure 资源管理器模板部署应用的指南](deploy-resource-manager-template.md)。

若要了解应用服务资源的 JSON 语法和属性，请参阅 [Microsoft.Web 资源类型](https://docs.microsoft.com/azure/templates/microsoft.web/allversions)。

| 部署应用 | 说明 |
|-|-|
| [应用服务计划和基本的 Linux 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-linux) | 部署为 Linux 配置的应用服务应用。 |
| [应用服务计划和基本的 Windows 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-windows) | 部署为 Windows 配置的应用服务应用。 |
| [链接到 GitHub 存储库的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-github-deploy)| 部署从 GitHub 提取代码的应用服务应用。 |
| [使用自定义部署槽位的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-custom-deployment-slots)| 使用自定义部署槽位/环境部署应用服务应用。 |
|**配置应用**| **说明** |
| [来自 Key Vault 的应用证书](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-certificate-from-key-vault)| 部署来自 Azure Key Vault 机密的应用服务应用证书并将其用于 TLS/SSL 绑定。 |
| [使用自定义域和 SSL 的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain-and-ssl)| 使用自定义主机名部署应用服务应用，并从 Key Vault 获取应用证书以用于 TLS/SSL 绑定。 |
| [使用 GoLang 扩展的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-with-golang)| 使用 Golang 站点扩展部署应用服务应用。 然后，可以在 Azure 中运行在 GoLang 上开发的 Web 应用程序。 |
| [使用 Java 8 和 Tomcat 8 的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-java-tomcat)| 部署已启用 Java 8 和 Tomcat 8 的应用服务应用。 然后，可以在 Azure 中运行 Java 应用程序。 |
|**保护应用**| **说明** |
| [与 Azure 应用程序网关集成的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-app-gateway-v2)| 部署应用服务应用和应用程序网关，并使用服务终结点和访问限制隔离流量。 |
|**使用连接资源的 Linux 应用**| **说明** |
| [Linux 上的使用 MySQL 的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-mysql) | 在 Linux 上部署使用 Azure Database for MySQL 的应用服务应用。 |
| [Linux 上的使用 PostgreSQL 的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-postgresql) | 在 Linux 上部署使用 Azure Database for PostgreSQL 的应用服务应用。 |
|**使用连接资源的应用**| **说明** |
| [使用 MySQL 的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-mysql)| 在 Windows 上部署使用 Azure Database for MySQL 的应用服务应用。 |
| [使用 PostgreSQL 的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-postgresql)| 在 Windows 上部署使用 Azure Database for PostgreSQL 的应用服务应用。 |
| [使用 Azure SQL 数据库中的数据库的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database)| 在“基本”服务级别部署应用服务应用和 Azure SQL 数据库中的数据库。 |
| [使用 Blob 存储连接的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-blob-connection)| 使用 Azure Blob 存储连接字符串部署应用服务应用。 然后，可以从该应用使用 Blob 存储。 |
| [使用用于 Redis 的 Azure 缓存的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-redis-cache)| 使用用于 Redis 的 Azure 缓存部署应用服务应用。 |
| [连接到后端 webapp 的应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-privateendpoint-vnet-injection)| 将两个 Web 应用（前端和后端）与 VNet 注入和专用终结点安全地连接在一起。 |
|**应用服务环境**| **说明** |
| [创建应用服务环境 v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-create) | 在虚拟网络中创建应用服务环境 v2。 |
| [使用 ILB 地址创建应用服务环境 v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-ilb-create/) | 使用专用的内部负载均衡器地址在虚拟网络中创建应用服务环境 v2。 |
| [为 ILB 应用服务环境或 ILB 应用服务环境 v2 配置默认的 SSL 证书](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-ase-ilb-configure-default-ssl) | 为 ILB 应用服务环境或 ILB 应用服务环境 v2 配置默认的 TLS/SSL 证书。 |
| | |

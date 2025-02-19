---
title: "SqlClient support for high availability, disaster recovery"
description: "Describes SqlClient support for high-availability, disaster recovery (AlwaysOn) availability groups."
ms.date: "08/15/2019"
ms.assetid: 61e0b396-09d7-4e13-9711-7dcbcbd103a0
ms.prod: sql
ms.prod_service: connectivity
ms.technology: connectivity
ms.topic: conceptual
author: David-Engel
ms.author: v-davidengel
ms.reviewer: v-kaywon
---
# SqlClient support for high availability, disaster recovery

[!INCLUDE[Driver_ADONET_Download](../../../includes/driver_adonet_download.md)]

This topic discusses Microsoft SqlClient Data Provider for SQL Server support for high-availability, disaster recovery -- AlwaysOn Availability Groups.  AlwaysOn Availability Groups feature was added to SQL Server 2012. For more information about AlwaysOn Availability Groups, see SQL Server Books Online.  
  
You can now specify the availability group listener of a (high-availability, disaster-recovery) availability group (AG) or SQL Server 2012 Failover Cluster Instance in the connection property. If a SqlClient application is connected to an AlwaysOn database that fails over, the original connection is broken and the application must open a new connection to continue work after the failover.  
  
If you are not connecting to an availability group listener or SQL Server 2012 Failover Cluster Instance, and if multiple IP addresses are associated with a hostname, SqlClient will iterate sequentially through all IP addresses associated with DNS entry. This can be time consuming if the first IP address returned by DNS server is not bound to any network interface card (NIC). When connecting to an availability group listener or SQL Server 2012 Failover Cluster Instance, SqlClient attempts to establish connections to all IP addresses in parallel and if a connection attempt succeeds, the driver will discard any pending connection attempts.  
  
> [!NOTE]
>  Increasing connection timeout and implementing connection retry logic will increase the probability that an application will connect to an availability group. Also, because a connection can fail because of a failover, you should implement connection retry logic, retrying a failed connection until it reconnects.  
  
The following connection properties are supported in the Microsoft SqlClient Data Provider for SQL Server:  
  
- `ApplicationIntent`  
  
- `MultiSubnetFailover`  
  
You can programmatically modify these connection string keywords with:  
  
- <xref:Microsoft.Data.SqlClient.SqlConnectionStringBuilder.ApplicationIntent%2A>  
  
- <xref:Microsoft.Data.SqlClient.SqlConnectionStringBuilder.MultiSubnetFailover%2A>  
  
## Connecting With MultiSubnetFailover  
Always specify `MultiSubnetFailover=True` when connecting to a SQL Server 2012 availability group listener or SQL Server 2012 Failover Cluster Instance. `MultiSubnetFailover` enables faster failover for all Availability Groups and or Failover Cluster Instance in SQL Server 2012 and will significantly reduce failover time for single and multi-subnet AlwaysOn topologies. During a multi-subnet failover, the client will attempt connections in parallel. During a subnet failover, will aggressively retry the TCP connection.  
  
The `MultiSubnetFailover` connection property indicates that the application is being deployed in an availability group or SQL Server 2012 Failover Cluster Instance and that SqlClient will try to connect to the database on the primary SQL Server instance by trying to connect to all the IP addresses. When `MultiSubnetFailover=True` is specified for a connection, the client retries TCP connection attempts faster than the operating system’s default TCP retransmit intervals. This enables faster reconnection after failover of either an AlwaysOn Availability Group or an AlwaysOn Failover Cluster Instance, and is applicable to both single- and multi-subnet Availability Groups and Failover Cluster Instances.  
  
For more information about connection string keywords in SqlClient, see <xref:Microsoft.Data.SqlClient.SqlConnection.ConnectionString%2A>.  
  
Specifying `MultiSubnetFailover=True` when connecting to something other than a availability group listener or SQL Server 2012 Failover Cluster Instance may result in a negative performance impact, and is not supported.  
  
Use the following guidelines to connect to a server in an availability group or SQL Server 2012 Failover Cluster Instance:  
  
- Use the `MultiSubnetFailover` connection property when connecting to a single subnet or multi-subnet; it will improve performance for both.  
  
- To connect to an availability group, specify the availability group listener of the availability group as the server in your connection string.  
  
- Connecting to a SQL Server instance configured with more than 64 IP addresses will cause a connection failure.  
  
- Behavior of an application that uses the `MultiSubnetFailover` connection property is not affected based on the type of authentication: SQL Server Authentication, Kerberos Authentication, or Windows Authentication.  
  
- Increase the value of `Connect Timeout` to accommodate for failover time and reduce application connection retry attempts.  
  
- Distributed transactions are not supported.  
  
 If read-only routing is not in effect, connecting to a secondary replica location will fail in the following situations:  
  
- If the secondary replica location is not configured to accept connections.  
  
- If an application uses `ApplicationIntent=ReadWrite` (discussed below) and the secondary replica location is configured for read-only access.  
  
<xref:Microsoft.Data.SqlClient.SqlDependency> is not supported on read-only secondary replicas.  
  
A connection will fail if a primary replica is configured to reject read-only workloads and the connection string contains `ApplicationIntent=ReadOnly`.  
  
## Upgrading to use multi-subnet clusters from database mirroring  
A connection error (<xref:System.ArgumentException>) will occur if the `MultiSubnetFailover` and `Failover Partner` connection keywords are present in the connection string, or if `MultiSubnetFailover=True` and a protocol other than TCP is used. An error (<xref:Microsoft.Data.SqlClient.SqlException>) will also occur if `MultiSubnetFailover` is used and the SQL Server returns a failover partner response indicating it is part of a database mirroring pair.  
  
If you upgrade a SqlClient application that currently uses database mirroring to a multi-subnet scenario, you should remove the `Failover Partner` connection property and replace it with `MultiSubnetFailover` set to `True` and replace the server name in the connection string with an availability group listener. If a connection string uses `Failover Partner` and `MultiSubnetFailover=True`, the driver will generate an error. However, if a connection string uses `Failover Partner` and `MultiSubnetFailover=False` (or `ApplicationIntent=ReadWrite`), the application will use database mirroring.  
  
The driver will return an error if database mirroring is used on the primary database in the AG, and if `MultiSubnetFailover=True` is used in the connection string that connects to a primary database instead of to an availability group listener.  
  
## Specifying application intent  
When `ApplicationIntent=ReadOnly`, the client requests a read workload when connecting to an AlwaysOn enabled database. The server will enforce the intent at connection time and during a USE database statement but only to an Always On enabled database.  
  
The `ApplicationIntent` keyword does not work with legacy, read-only databases.  
  
A database can allow or disallow read workloads on the targeted AlwaysOn database. (This is done with the `ALLOW_CONNECTIONS` clause of the `PRIMARY_ROLE` and `SECONDARY_ROLE`Transact-SQL statements.)  
  
The `ApplicationIntent` keyword is used to enable read-only routing.  
  
## Read-only routing  
Read-only routing is a feature that can ensure the availability of a read only replica of a database. To enable read-only routing:  
  
- You must connect to an Always On Availability Group availability group listener.  
  
- The `ApplicationIntent` connection string keyword must be set to `ReadOnly`.  
  
- The Availability Group must be configured by the database administrator to enable read-only routing.  
  
It is possible that multiple connections using read-only routing will not all connect to the same read-only replica. Changes in database synchronization or changes in the server's routing configuration can result in client connections to different read-only replicas. To ensure that all read-only requests connect to the same read-only replica, do not pass an availability group listener to the `Data Source` connection string keyword. Instead, specify the name of the read-only instance.  
  
Read-only routing may take longer than connecting to the primary because read only routing first connects to the primary and then looks for the best available readable secondary. Because of this, you should increase your login timeout.  
  
## Next steps
- [SQL Server features and ADO.NET](sql-server-features-adonet.md)

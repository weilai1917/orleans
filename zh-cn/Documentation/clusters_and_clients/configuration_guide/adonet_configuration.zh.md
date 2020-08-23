# 数据库配置

下面的段落包含链接SQL脚本，以配置您的数据库，以及相应的ADO.Net不变性，用于配置ADO.Net Providers in Orleans。这些脚本如果需要扩充，将被关闭。

## 聚类

| 数据库 | 脚本 | Nuget包装 | 不变性 |
| --- | --- | ------- | --- |
| SQL服务器 | [SQLServer-Clustering.sql](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Clustering.AdoNet/SQLServer-Clustering.sql) | [System.Data.SqlClient](https://www.nuget.org/packages/System.Data.SqlClient/) | System.Data.SqlClient |
| 玛丽亚 | [MySQL-Clustering.sql](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Clustering.AdoNet/MySQL-Clustering.sql) | [MySql.Data](https://www.nuget.org/packages/MySql.Data/) | MySql.Data.MySqlClient |
| 后格雷斯克尔 | [PostgreSQL-Clustering.sql](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Clustering.AdoNet/PostgreSQL-Clustering.sql) | [中华人民共和国](https://www.nuget.org/packages/Npgsql/) | 中华人民共和国 |
| 神谕 | [神谕集群](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Clustering.AdoNet/Oracle-Clustering.sql) | [ODP.net](https://www.nuget.org/packages/Oracle.ManagedDataAccess/) | oracle.dataaccess.client文件 |

## 坚持不懈

| 数据库 | 脚本 | NuGET包 | ADO.NET不变量 |
| --- | --- | ------ | ---------- |
| SQL Server | [sqlserver-persistence.sql数据库](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/SQLServer-Persistence.sql) | [system.data.sqlclient系统](https://www.nuget.org/packages/System.Data.SqlClient/) | system.data.sqlclient系统 |
| MySQL/马里亚行 | [mysql-persistence.sql文件](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/MySQL-Persistence.sql) | [mysql.数据](https://www.nuget.org/packages/MySql.Data/) | MySql.Data.MySqlClient |
| PostgreSQL公司 | [PostgreSQL-持久性.sql](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/PostgreSQL-Persistence.sql) | [NPGSQL公司](https://www.nuget.org/packages/Npgsql/) | NPGSQL公司 |
| 甲骨文公司 | [Oracle持久性.sql](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Oracle-Persistence.sql) | [网](https://www.nuget.org/packages/Oracle.ManagedDataAccess/) | oracle.dataaccess.client文件 |

## 提醒

| 数据库 | 脚本 | NuGET包 | ADO.NET不变量 |
| --- | --- | ------ | ---------- |
| SQL Server | [sqlserver-reminders.sql提示](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Reminders.AdoNet/SQLServer-Reminders.sql) | [system.data.sqlclient系统](https://www.nuget.org/packages/System.Data.SqlClient/) | system.data.sqlclient系统 |
| MySQL/马里亚行 | [mysql-reminders.sql文件](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Reminders.AdoNet/MySQL-Reminders.sql) | [MySQL数据](https://www.nuget.org/packages/MySql.Data/) | MySql.Data.MySqlClient |
| PostgreSQL公司 | [PostgreSQL提醒.sql](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Reminders.AdoNet/PostgreSQL-Reminders.sql) | [NPGSQL公司](https://www.nuget.org/packages/Npgsql/) | NPGSQL公司 |
| 神谕。 | [Oracle提醒.sql](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Reminders.AdoNet/Oracle-Reminders.sql) | [网](https://www.nuget.org/packages/Oracle.ManagedDataAccess/) | oracle.dataaccess.client文件 |

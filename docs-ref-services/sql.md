---
title: Python용 Azure SQL Database 라이브러리
description: ODBC 드라이버와 pyodbc를 사용하여 Azure SQL 데이터베이스에 연결하거나 관리 API로 Azure SQL 인스턴스를 관리합니다.
author: lisawong19
ms.author: routlaw
manager: routlaw
ms.date: 01/09/2018
ms.topic: reference
ms.devlang: python
ms.service: sql-database
ms.openlocfilehash: 9b8a5b120425fc600f34c1e4c4456b0888814fe8
ms.sourcegitcommit: 46bebbf5dd558750043ce5afadff2ec3714a54e6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/03/2019
ms.locfileid: "67534208"
---
# <a name="azure-sql-database-libraries-for-python"></a>Python용 Azure SQL Database 라이브러리

## <a name="overview"></a>개요

pyodbc [ODBC 데이터베이스 드라이버](https://github.com/mkleehammer/pyodbc/wiki/Drivers-and-Driver-Managers)를 사용하여 Python의 [Azure SQL Database](/azure/sql-database/sql-database-technical-overview)에 저장된 데이터로 작업합니다. [빠른 시작](https://docs.microsoft.com/azure/sql-database/sql-database-connect-query-python)을 보면서 Azure SQL 데이터베이스에 연결하고 Transact-SQL 문을 사용하여 데이터를 쿼리하고 pyodbc를 사용하여 [샘플](https://github.com/mkleehammer/pyodbc/wiki/Getting-started)을 시작합니다.

## <a name="install-odbc-driver-and-pyodbc"></a>ODBC 드라이버 및 pyodbc 설치

```bash
pip install pyodbc
```
Python 및 데이터베이스 통신 라이브러리 설치에 대한 [세부 정보](https://docs.microsoft.com/azure/sql-database/sql-database-connect-query-python#prerequisites)입니다.

## <a name="connect-and-execute-a-sql-query"></a>SQL 쿼리 연결 및 실행

### <a name="connect-to-a-sql-database"></a>SQL 데이터베이스에 연결

```python
import pyodbc

server = 'your_server.database.windows.net'
database = 'your_database'
username = 'your_username'
password = 'your_password'
driver= '{ODBC Driver 13 for SQL Server}'

cnxn = pyodbc.connect('DRIVER='+driver+';PORT=1433;SERVER='+server+';PORT=1443;DATABASE='+database+';UID='+username+';PWD='+ password)
cursor = cnxn.cursor()
```

### <a name="execute-a-sql-query"></a>SQL 쿼리 실행

```python
cursor.execute("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName FROM [SalesLT].[ProductCategory] pc JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid")
row = cursor.fetchone()
while row:
    print (str(row[0]) + " " + str(row[1]))
    row = cursor.fetchone()
```

> [!div class="nextstepaction"]
> [pyodbc 샘플](https://github.com/mkleehammer/pyodbc/wiki/Getting-started)

## <a name="connecting-to-orms"></a>ORM에 연결

pyodbc는 [SQLAlchemy](https://docs.sqlalchemy.org/en/latest/dialects/mssql.html?highlight=pyodbc#module-sqlalchemy.dialects.mssql.pyodbc) 및 [Django](https://github.com/lionheart/django-pyodbc/) 등의 다른 ORM과 함께 작동합니다. 

## <a name="management-apipythonapioverviewazuresqlmanagement"></a>[관리 API](/python/api/overview/azure/sql/management)

관리 API를 사용하여 구독의 Azure SQL Database 리소스를 만들고 관리합니다. 

```bash
pip install azure-common
pip install azure-mgmt-sql
pip install azure-mgmt-resource
```

## <a name="example"></a>예

SQL Database 리소스를 만들고 방화벽 규칙을 사용하여 IP 주소 범위에 대한 액세스를 제한합니다.

```python
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.resource import ResourceManagementClient
from azure.mgmt.sql import SqlManagementClient

RESOURCE_GROUP = 'YOUR_RESOURCE_GROUP_NAME'
LOCATION = 'eastus'  # example Azure availability zone, should match resource group
SQL_SERVER = 'yourvirtualsqlserver'
SQL_DB = 'YOUR_SQLDB_NAME'
USERNAME = 'YOUR_USERNAME'
PASSWORD = 'YOUR_PASSWORD'

# create resource client
resource_client = get_client_from_cli_profile(ResourceManagementClient)
# create resource group
resource_client.resource_groups.create_or_update(RESOURCE_GROUP, {'location': LOCATION})

sql_client = get_client_from_cli_profile(SqlManagementClient)

# Create a SQL server
server = sql_client.servers.create_or_update(
    RESOURCE_GROUP,
    SQL_SERVER,
    {
        'location': LOCATION,
        'version': '12.0', # Required for create
        'administrator_login': USERNAME, # Required for create
        'administrator_login_password': PASSWORD # Required for create
    }
)

# Create a SQL database in the Basic tier
database = sql_client.databases.create_or_update(
    RESOURCE_GROUP,
    SQL_SERVER,
    SQL_DB,
    {
        'location': LOCATION,
        'collation': 'SQL_Latin1_General_CP1_CI_AS',
        'create_mode': 'default',
        'requested_service_objective_name': 'Basic'
    }
)

# Open access to this server for IPs
firewall_rule = sql_client.firewall_rules.create_or_update(
    RESOURCE_GROUP,
    SQL_DB,
    "firewall_rule_name_123.123.123.123",
    "123.123.123.123", # Start ip range
    "167.220.0.235"  # End ip range
)
```
> [!div class="nextstepaction"]
> [관리 API 탐색](/python/api/overview/azure/sql/management)


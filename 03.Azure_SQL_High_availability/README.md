
## General Purpose or Standard 
### 01. Geo-Replication
Azure SQL에서는 손쉽게 전 세계적으로 Replication을 구성할 수 있습니다.
![sqlgeorep](https://docs.microsoft.com/ko-kr/azure/azure-sql/database/media/active-geo-replication-overview/geo-replication.png)  
아래와 같이 Japan east에 Azure CLI로 Replication을 구성 합니다.
```powershell
$repLocation="japaneast"
$repServerName="*myservername"
az sql server create --name $repServerName --resource-group $resourceGroup --location $repLocation --admin-user $userName --admin-password $password
az sql db replica create --name $dbName --partner-server $repServerName --resource-group $resourceGroup --server $serverName
```

각 database에서 아래의 쿼리를 통해 해당 db에서의 updateability(READ_ONLY, READ_WRITE)를 확인할 수 있습니다.

```sql
-- 현재 접속이 읽기 전용 복제본인지 확인
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Updateability')
```

### 02. Failover Group
Geo-Replication에서는 slave node를 master로 fail-over 하려면 수동으로 진행 가능 합니다.
지역간의 fail-over를 자동으로 관리하기 위해서는 Failover Group을 사용하여야 합니다.

![azsqlfog](https://docs.microsoft.com/ko-kr/azure/azure-sql/database/media/auto-failover-group-overview/auto-failover-group.png)

구성을 위해서 아래 Azure CLI로 Replication이 진행된 primary (master), seconday (slave) 서버를 Failover Group에 추가 합니다.
```powershell
$fogName="*myFogName"
# 신규 replica를 failover group으로 생성
az sql failover-group create --name $fogName --partner-server $repServerName  --resource-group $resourceGroup --server $serverName
```

FOG (Failover group)을 사용하면 각 SQL Server의 endpoint를 사용하지 않아도 리스너를 통하여 primary와 secondary의 endpoint를 사용할 수 있습니다.

![fogendpoint](https://azmyhanson.blob.core.windows.net/azcon/01_fogendpoint.jpg)

## Business Critical or Premium
Azure SQL Database Business critical 혹은 Premium tier 에서는 별도의 비용 없이 Zone Redundant (지원하는 지역에 한함), Failover Group 및 Read-Only Replica를 사용할 수 있습니다.

![sqlbc](https://docs.microsoft.com/en-us/azure/azure-sql/database/media/read-scale-out/business-critical-service-tier-read-scale-out.png)

Read-Only Replica를 테스트 하기 위해 SSMS를 통하여 기본 endpoint로 접속하여 아래 쿼리를 확인 합니다.
```sql
-- 현재 접속이 읽기 전용 복제본인지 확인
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Updateability')
```
Read-Only로 접속하기 위해 Connection String에 ApplicationIntent=ReadOnly; 추가합니다.
SSMS에서 Connection String을 추가하기 위해서는 옵션을 클릭 후 추가 연결 매개변수에 추가합니다.

![ssms00](https://azmyhanson.blob.core.windows.net/azcon/00_ssms_connection.jpg)




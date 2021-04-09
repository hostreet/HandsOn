
### 01. Time zone 변경
Azure SQL은 Timezone을 선택할 수 없으며 UTC 0으로 제공 됩니다 (Managed Instance는 Time zone 선택 가능)  
CURRENT_TIMESTAMP, GETDATE() 등 날짜 관련 함수를 사용하면 시간값이 UTC 0로 반환 되는 것을 확인할 수 있습니다
```sql
SELECT CURRENT_TIMESTAMP, GETDATE()
```
만일 신규 서비스가 아닌 기존에 한국 시간 기준으로 GETDATE()등을 사용해왔던 서비스라면 마이그레이션 시 시간은 변환해야 합니다  
변환하는 방법은 DB에서 GETDATE()등을 사용하지 않고, DB에 쿼리하는 서버 시간 기준으로 변경하거나  
별도의 사용자 함수를 추가해서 쿼리들을 변경해야 합니다  
SSMS에서 아래 Function을 추가하고 사용하는 방법을 확인 합니다  
```sql
CREATE FUNCTION FN_GETDATE()
RETURNS DATETIME
AS
BEGIN
     DECLARE @D AS datetimeoffset
     SET @D = CONVERT(datetimeoffset, GETDATE()) AT TIME ZONE 'Korea Standard Time'
     RETURN CONVERT(datetime, @D);
END
SELECT dbo.FN_GETDATE()
```
### 02. Elastic Query 
Azure SQL에서는 데이터베이스 간 Join 혹은 조회 및 Linked Server를 사용할 수 없습니다  
대신 Elastic Query를 통하여 원격지의 데이터를 조회할 수 있습니다  
다음 HandsOn에서는 기존 데이터베이스 서버에서 새로운 데이터베이스를 연결 후 두 데이터베이스간 Join하여 쿼리 하는 방법을 알아 봅니다  

```powershell
$newDBName="*myNewDBName"
az sql db create -g $resourceGroup -s $serverName -n $dbName --collation $collation --sample-name AdventureWorksLT -e GeneralPurpose -f Gen4 -c 1
az sql db create -g $resourceGroup -s $serverName -n $newDBName --collation $collation --sample-name AdventureWorksLT -e GeneralPurpose -f Gen5 -c 2
# Allow Azure service
## 시작과 끝 아이피를 0.0.0.0 으로 지정하면 Azure 서비스 및 리소스가 서버에 엑세스 할수 있도록 허용 됩니다
az sql server firewall-rule create -g $resourceGroup -s $serverName -n allowazs --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```
- 위에 생성한 new DB에 접속
- 데이터베이스 서버에서 Azure Service 방문 허용 True로 변경 (azure portal)
- master key 생성
- Database Scoped Credential 생성
- External Data Source 생성
- 외부 테이블과 동일한 스키마의 외부 테이블 생성
```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<password>' ;
CREATE DATABASE SCOPED CREDENTIAL SQL_Credential
WITH
  IDENTITY = '<username>' ,
  SECRET = '<password>' ;
CREATE EXTERNAL DATA SOURCE MyElasticDBQueryDataSrc
WITH
  ( TYPE = RDBMS ,
    LOCATION = '<server_name>.database.windows.net' ,
    DATABASE_NAME = 'Customers' ,
    CREDENTIAL = SQL_Credential
  ) ;
-- 외부 테이블과 동일한 스키마를 가진 테이블을 생성 합니다
CREATE EXTERNAL TABLE [dbo].[ExtCustomer](
	[CustomerID] [int]  NOT NULL,
	[NameStyle] [dbo].[NameStyle] NOT NULL,
	[Title] [nvarchar](8) NULL,
	[FirstName] [dbo].[Name] NOT NULL,
	[MiddleName] [dbo].[Name] NULL,
	[LastName] [dbo].[Name] NOT NULL,
	[Suffix] [nvarchar](10) NULL,
	[CompanyName] [nvarchar](128) NULL,
	[SalesPerson] [nvarchar](256) NULL,
	[EmailAddress] [nvarchar](50) NULL,
	[Phone] [dbo].[Phone] NULL,
	[PasswordHash] [varchar](128) NOT NULL,
	[PasswordSalt] [varchar](10) NOT NULL,
	[rowguid] [uniqueidentifier] NOT NULL,
	[ModifiedDate] [datetime] NOT NULL)
WITH
( 
	DATA_SOURCE = MyElasticDBQueryDataSrc,
	SCHEMA_NAME = 'SalesLT',
	OBJECT_NAME = 'Customer'
)
SELECT TOP 100 *
FROM [dbo].[ExtCustomer] a
	INNER JOIN SalesLT.CustomerAddress b
	ON a.CustomerID = b.CustomerID
```



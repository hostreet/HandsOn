
## 01. Azure SQL Database 생성 (General purpose - 2vCore)

portal에서 [sql database]를 클릭하여 생성버튼을 누른 뒤 아래와 같이 입력합니다.

![sqldatabase_create](https://user-images.githubusercontent.com/82139935/114138879-3a04fd80-9949-11eb-9476-8c1f506c655f.PNG)

1. resource group : 이전 VM을 생성할 때 사용했던 리소스 그룹을 선택합니다.
2. database name : 신규 생성될 sql server 하위의 데이터베이스 이름(sql server 내에서만 고유하면 됨)을 기입합니다.
3. server : [server_name].database.windows.net 와 같은 endpoint가 생성되므로 고유한 이름이여야 합니다. (아래 Business Critical생성 시 동일한 server 사용 예정)
4. server name : 고유한 이름의 endpoint 명을 지정해줍니다.
5. 접속할 id/pw를 지정해줍니다.
6. 서버 생성 후 데이터 베이스 구성을 클릭하게되면 서비스에 알맞은 SKU를 선택할 수 있습니다.

![sqldatabase_sku](https://user-images.githubusercontent.com/82139935/114138882-3b362a80-9949-11eb-9f01-06b356950130.PNG)

7. 검토 + 만들기를 눌러 생성을 완료합니다.



### cli 통해서 진행 할 경우

```powershell
# 파라미터 앞에 *이 붙은 항목은 필수 변경
$resourceGroup="rg-adstest"
$location="koreacentral"
$serverName="*myservername"
$dbName="*myDbname"
$userName="*myUsername"
$password="*myPassword"
$collation="Korean_Wansung_CI_AS"
az sql server create -l $location -g $resourceGroup -n $serverName -u $userName -p $password
az sql db create -g $resourceGroup -s $serverName -n $dbName --collation $collation --sample-name AdventureWorksLT -e GeneralPurpose -f Gen5 -c 2
```
![azsqlgp](https://docs.microsoft.com/ko-kr/azure/azure-sql/database/media/high-availability-sla/general-purpose-service-tier.png)


## 02. Azure SQL Database 생성 (Business critical - 2vCore)

위에서 생성한 General Purpose와 동일하게 진행하되 6번 내용의 SKU를 아래와 같이 Business Critical로 변경하여 생성해 줍니다.
(sql server의 경우 의에서 GP로 리소스가 생성되면 같은 server하위에 database를 생성하시거나 별도의 server로 생성하셔도 무방합니다.)

![sqldatabase_sku2](https://user-images.githubusercontent.com/82139935/114138883-3bcec100-9949-11eb-9f0f-6e1751ba3d39.PNG)

### cli 통해서 진행 할 경우

```powershell
$location="koreacentral"
# 기존 General purpose와 다른 변수 입력
$bcServerName="*myservername"
$bcDbName="*mydbname"
az sql server create -l $location -g $resourceGroup -n $bcServerName -u $userName -p $password
az sql db create -g $resourceGroup -s $bcServerName -n $bcDbName --collation $collation --sample-name AdventureWorksLT -e BusinessCritical  -f Gen5 -c 2 
```

![azsqlbc](https://docs.microsoft.com/ko-kr/azure/azure-sql/database/media/high-availability-sla/business-critical-service-tier.png)



## 03. Check
위의 과정대로 모두 생성한 뒤에 Azure portal에서 sql server를 쳐보면 아래와 같이 sql server 하위에 GP(General Purpose), BC(Business Critical) 2개의 database가 생성되어 있습니다.




### 01. 외부 접속 허용
만일 회사나 집 등 외부에서 접속하기 위해서는 public ip를 접속 가능하도록 변경 합니다.  
Azure SQL Database는 외부 접속을 서버수준과 데이터베이스 수준에서 허용할 수 있습니다.  


#### 서버 수준 IP 방화벽 규칙
서버에서 관리되는 모든 데이터베이스에 접근을 허용 합니다.  
Azure Portal,Azure CLI,Azure Powershell 및 SSMS 등 데이터베이스에서 허용할 수 있습니다.
### [Portal] 통해서 생성한 VM의 IP접속에 대한 허용을 진행합니다.

이미지넣기!!

1. sql server(database아님)를 선택한 상태에서 왼쪽 보안탭의 방화벽 및 가상 네트워크를 클릭합니다.
2. 현재 클라이언트 IP 주소가 표기되는데 우리는 VM에서 SSMS를 통해 접속 할 것이므로 이전에 생성한 VM의 IP를 기입해야합니다.
3. 자신이 구분할 수 있는 규칙의 이름과 시작 IP, 종료 IP를 순서대로 기입해 줍니다.
4. 위 저장을 눌러 변경사항을 업데이트합니다.

#### [SSMS] 서버 수준 IP 방화벽 규칙
서버에서 관리되는 모든 데이터베이스에 접근을 허용 합니다.
Azure Portal,Azure CLI,Azure Powershell 및 SSMS 등 데이터베이스에서 허용할 수 있습니다.

SSMS에서 실행 (방화벽 추가)  
```sql
-- 아이피 수정 필요
-- master database에서 수행
EXECUTE sp_set_firewall_rule N'allowmyip', '*0.0.0.0', '*0.0.0.0';  
```

SSMS에서 방화벽 확인
```sql
-- master database에서 수행
select * from sys.firewall_rules;
```

SSMS에서 실행 (방화벽 제거)  
```sql
-- master database에서 수행
EXEC sp_delete_firewall_rule N'allowmyip'
```

#### 데이터베이스 수준 IP 방화벽 규칙
#### [SSMS] 데이터베이스 수준 IP 방화벽 규칙
특정 데이터베이스에 접속을 허용 합니다.
Azure SQL 에서 SSMS로 허용 합니다.

SSMS에서 실행 (방화벽 추가)  
```sql
-- 아이피 수정 필요
-- 개별 database에서 수행
EXECUTE sp_set_database_firewall_rule N'allowmyip', '*0.0.0.0', '*0.0.0.0';  
```

SSMS에서 확인
```sql
-- 개별 database에서 수행
select * from sys.database_firewall_rules;
```

SSMS에서 실행 (방화벽 제거)  
```sql
-- 개별 database에서 수행
EXEC sp_delete_database_firewall_rule N'allowmyip'
```

### 02. 방화벽 및 Service Endpoint 추가
기존 생성한 VM 에서 Azure SQL로 접속하기 위해서는 VM에 속한 VNet - Subnet에서 Service endpoint를 추가한 뒤 해당 서브넷을 허용 하여야 합니다.
Subnet에서 Service endpoint 추가 시 해당 VNet이 잠시동안 가동 중지될 수 있으니 주의하여야 합니다.
  
만일 당장 VNet이 잠시 중단 되는 것을 원하지 않는다면 IgnoreMissingVNetServiceEndpoint 플래그를 활성화하여 
Service endpoint를 무시하여 VNet이 가동 중지 되는 것을 방지한 후 가능한 시점에 Service Endpoint를 추가할 수 있습니다.

![servcie_endpoint](https://user-images.githubusercontent.com/82139935/114144161-fe216680-994f-11eb-9c58-84aea9786bb2.png)


### 03. Private Link for Azure SQL Database
서비스 엔드 포인트에는 몇 가지 제한이나 단점이 있습니다.
서비스 엔드 포인트에 대한 트래픽이 여전히 가상 네트워크 외부로 나가며 Azure PaaS 리소스가 여전히 해당 공용 주소에서 액세스되고 있습니다.
VPN 또는 Express Route와 같은 연결로 Azure Virtual Network를 통해 들어오는 트래픽에만 사용할 수 있습니다.  
온 프레미스 리소스에 대한 액세스를 허용하려면 해당 퍼블릭 IP도 허용해야합니다.  

![private_endpoint](https://user-images.githubusercontent.com/82139935/114144167-ff529380-994f-11eb-80fc-27ca7de27169.png)


### 04. Service Endpoint vs Private Link
Service Endpoint
- VNET에 PaaS resource 접근에 대한 권한을 위임
- PIP

Private Link
- Service에 대한 트래픽이 항상 VNET내에 있음

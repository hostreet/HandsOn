
## Webinar Demo Architecture


![webinar_architecture3](https://user-images.githubusercontent.com/82139935/114117694-3742e200-9922-11eb-9f4b-e1be3050e6e0.PNG)




### 01. Azure SQL CeateResource
### 02. Azure SQL Fiewalls Networks
### 03. Azure SQL High availability
### 04. Azure SQL Limit
### 05. Azure SQL Monitoring
##              



### 사전준비
Azure SQL Database HOL 진행을 위해  
CLI 및 SSMS (SQL Server Management Studio) 다운로드 및 설치를 진행 합니다.

#### Azure CLI 설치
[Azure CLI Download](https://aka.ms/installazurecliwindows)  
수동 설치 혹은 PowerShell을 사용하여 Azure CLI를 설치할 수도 있습니다.
관리자 권한으로 PowerShell을 시작하고 다음 명령을 실행합니다.

#### Azure CLI Login
먼저 자신의 계정으로 로그인 합니다.
만일 자신의 계정에 구독이 여러개 있다면 테스트에서 사용할 구독을 입력 합니다.
```powershell
az login 
az account set --subscription "{your subscription}"
az account show
```

## Azure Virtual Machine 생성

이미지!!


생성된 vm resource를 클릭해보면 공용 IP 주소라고 되어있는 부분을 통해 원격접속 합니다.


#### HandsOn 진행을 위한 Tools 설치파일 다운로드 및 설치
#### Client Tool(SSMS) 설치
자동 설치

생성된 VM 접속 후 Powershell

```powershell
# SQL Server Management Studio 설치
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Invoke-WebRequest -Uri "https://aka.ms/ssmsfullsetup" -OutFile .\SSMS-Setup-KOR.exe; 
$Parms = " /Install /Quiet /Norestart /Logs log.txt"
$Prms = $Parms.Split(" ")
& .\SSMS-Setup-KOR.exe $Prms | Out-Null
```
자동 설치 실패 시 수동 설치  
[SSMS](https://docs.microsoft.com/ko-kr/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15)  

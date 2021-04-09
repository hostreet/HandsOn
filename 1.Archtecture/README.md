Azure SQL Database HOL 진행을 위해  
테스트용 VM 배포 및 SSMS (SQL Server Management Studio) 다운로드 및 설치를 진행 합니다

### 01. Azure CLI 설치
[Azure CLI Download](https://aka.ms/installazurecliwindows)  
수동 설치 혹은 PowerShell을 사용하여 Azure CLI를 설치할 수도 있습니다   
관리자 권한으로 PowerShell을 시작하고 다음 명령을 실행합니다  
### 01. Azure VM 생성
여러 테스트 환경(mac, windows...)을 일괄적으로 통일 시키기 위해서 azure 상에서 VM을 생성하고 해당 VM에 MSSQL client Tool(SSMS)를 설치하여 HOL진행을 합니다.

```powershell
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; 
Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; 
rm .\AzureCLI.msi
 ```

### 02. Azure CLI Login
먼저 자신의 계정으로 로그인 합니다  
만일 자신의 계정에 구독이 여러개 있다면 테스트에서 사용할 구독을 입력 합니다  
```powershell
az login 
az account set --subscription "{your subscription}"
az account show
```

### 03. Azure VM 생성
```powershell
# Parameters 임의로 구성되었으며 수정 가능
# 파라미터 앞에 *이 붙은 항목은 필수 변경
$location="koreacentral"
$resourceGroup="rg-adstest"
$vnetName="vnet-adstest"
$subnetName="subnet-adstest"
$publicIPName="pip-adstestvm"
$nsgName="nsg-adstest"
$nicName="nic-adstest"
$vmName="vm-adstest"
$vmSize="Standard_D4s_v3"
# 계정 정보 입력
$AdminUserName="*username"
$AdminPassword="*password"
# Create a resource group.
az group create --name $resourceGroup --location $location
# Create a virtual network.
az network vnet create --resource-group $resourceGroup --name $vnetName --subnet-name $subnetName
# Create a public IP address.
az network public-ip create --resource-group $resourceGroup --name $publicIPName
# Create a network security group.
az network nsg create --resource-group $resourceGroup --name $nsgName
# Create a virtual network card and associate with public IP address and NSG.
az network nic create --resource-group $resourceGroup --name $nicName `
--vnet-name $vnetName --subnet $subnetName --network-security-group $nsgName --public-ip-address $publicIPName
# Create a virtual machine. 
az vm create --resource-group $resourceGroup --name $vmName --location $location --nics $nicName --image win2016datacenter `
--admin-username $AdminUserName --admin-password $AdminPassword --size $vmSize
# Open port 3389 to allow RDP traffic to host.
az vm open-port --port 3389 --resource-group $resourceGroup --name $vmName
```
<img src = "../images/mssql_image01.PNG" width="60%">

생성된 vm resource를 클릭해보면 공용 IP 주소라고 되어있는 부분을 통해 원격접속 합니다.

### 04. HandsOn 진행을 위한 Tools 설치파일 다운로드 및 설치
### 02. Client Tool(SSMS) 설치
자동 설치  
생성된 VM 접속 후 Powershell 

Azure SQL Database HOL 진행을 위해  
테스트용 VM 배포 및 SSMS (SQL Server Management Studio) 다운로드 및 설치를 진행 합니다

### 01. Azure CLI 설치
[Azure CLI Download](https://aka.ms/installazurecliwindows)  
수동 설치 혹은 PowerShell을 사용하여 Azure CLI를 설치할 수도 있습니다   
관리자 권한으로 PowerShell을 시작하고 다음 명령을 실행합니다  
### 01. Azure VM 생성
여러 테스트 환경(mac, windows...)을 일괄적으로 통일 시키기 위해서 azure 상에서 VM을 생성하고 해당 VM에 MSSQL client Tool(SSMS)를 설치하여 HOL진행을 합니다.

```powershell
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; 
Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; 
rm .\AzureCLI.msi
 ```

### 02. Azure CLI Login
먼저 자신의 계정으로 로그인 합니다  
만일 자신의 계정에 구독이 여러개 있다면 테스트에서 사용할 구독을 입력 합니다  
```powershell
az login 
az account set --subscription "{your subscription}"
az account show
```

### 03. Azure VM 생성
```powershell
# Parameters 임의로 구성되었으며 수정 가능
# 파라미터 앞에 *이 붙은 항목은 필수 변경
$location="koreacentral"
$resourceGroup="rg-adstest"
$vnetName="vnet-adstest"
$subnetName="subnet-adstest"
$publicIPName="pip-adstestvm"
$nsgName="nsg-adstest"
$nicName="nic-adstest"
$vmName="vm-adstest"
$vmSize="Standard_D4s_v3"
# 계정 정보 입력
$AdminUserName="*username"
$AdminPassword="*password"
# Create a resource group.
az group create --name $resourceGroup --location $location
# Create a virtual network.
az network vnet create --resource-group $resourceGroup --name $vnetName --subnet-name $subnetName
# Create a public IP address.
az network public-ip create --resource-group $resourceGroup --name $publicIPName
# Create a network security group.
az network nsg create --resource-group $resourceGroup --name $nsgName
# Create a virtual network card and associate with public IP address and NSG.
az network nic create --resource-group $resourceGroup --name $nicName `
--vnet-name $vnetName --subnet $subnetName --network-security-group $nsgName --public-ip-address $publicIPName
# Create a virtual machine. 
az vm create --resource-group $resourceGroup --name $vmName --location $location --nics $nicName --image win2016datacenter `
--admin-username $AdminUserName --admin-password $AdminPassword --size $vmSize
# Open port 3389 to allow RDP traffic to host.
az vm open-port --port 3389 --resource-group $resourceGroup --name $vmName
```
<img src = "../images/mssql_image01.PNG" width="60%">

생성된 vm resource를 클릭해보면 공용 IP 주소라고 되어있는 부분을 통해 원격접속 합니다.

### 04. HandsOn 진행을 위한 Tools 설치파일 다운로드 및 설치
### 02. Client Tool(SSMS) 설치
자동 설치  
생성된 VM 접속 후 Powershell 

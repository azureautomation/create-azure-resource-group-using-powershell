Create Azure Resource Group using PowerShell
============================================

            
This Script was created to install Resource group including storage account, virtual network & storage content . The belowscript is for installing Azure Resource group.


 



PowerShell
Edit|Remove
powershell
##########################################################################################################
<#
.SYNOPSIS
    Create Lab resource group, storage account, virtual network and VMs
    
.DESCRIPTION
    This script will create the following components
    -	Resource group: it will contain all VMs, storage account, virtual network and other resources required for the lab.
    -    you can edit on labPrefix, labnumber, labsubnet & Ip address with name as you want

.NOTES
    THIS CODE-SAMPLE IS PROVIDED 'AS IS' WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED 
    OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF MERCHANTABILITY AND/OR 
    FITNESS FOR A PARTICULAR PURPOSE.


#>
##########################################################################################################
###############Install Azure Module#########################
Install-Module -Name AzureRM -force
Import-Module -Name AzureRM
Install-Module -Name AzureRM.compute
Import-Module -Name AzureRM.compute

# Connect to Azure

Login-AzurermAccount

# Select Azure Subscription
$sub = Get-AzureRMsubscription
Select-AzureRmSubscription -SubscriptionId $sub[0].SubscriptionId

# Set values for new resource group, storage account and vnet
$labPrefix = 'Mlab'
$labnumber = '2017'
$labsubnet = '55'
$rgName = $labPrefix + $labnumber #New resource group name
$locName = 'West Europe' # Loation of new resource group
$saName = $rgName.Replace('-','').tolower() # Storage account name (for new resource group)
$saType='Standard_LRS' # Storage account type
$vnetname = $rgName # New virtual network name
$subnetIndex=0 # Frontend subnet ID
$frontendsubnetname = $rgName + '-FE' # Frontend subnet name
$backendsubnetname = $rgName + '-BE' # Backtend subnet name
$vnetsubnet = '10.$labsubnet.0.0/16' # Virtual network suffix
$frontendsubnetrange = '10.$labsubnet.1.0/24' # Frontend subnet range
$backendsubnetrange = '10.$labsubnet.2.0/24' # Backtend subnet name
$NSGName = $rgName + '-NSG'

## Create initial resources
# create resource group
New-AzureRmResourceGroup -Name $rgName -Location $locName | Out-Null
write-output '1/14 - the resource group has been created successfully'
# create storage account
New-AzureRmStorageAccount -Name $saName -ResourceGroupName $rgName –Type $saType -Location $locName | Out-Null
write-output '2/14 - the storage account has been created successfully'
# get storage account key
$key1 = (Get-AzureRmStorageAccountKey -Name $saName -ResourceGroupName $rgName).Value[0]
# create storage context
$storagecontext = New-AzureStorageContext -StorageAccountName $saname -StorageAccountKey $key1
# create a container called scripts
New-AzureStorageContainer -Name 'scripts' -Context $storagecontext -Permission BLOB  | Out-Null
write-output '3/14 - Scripts Container has been created successfully'
New-AzureStorageContainer -Name 'baseimage' -Context $storagecontext -Permission BLOB | Out-Null
write-output '4/14 - Base Image Container has been created successfully'


########################################################## Virtual network config ##############################################################

$frontendSubnet=New-AzureRmVirtualNetworkSubnetConfig -Name $frontendsubnetname -AddressPrefix $frontendsubnetrange
$backendSubnet=New-AzureRmVirtualNetworkSubnetConfig -Name $backendsubnetname -AddressPrefix $backendsubnetrange
$vnet = New-AzureRmVirtualNetwork -Name $vnetname -ResourceGroupName $rgName -Location $locName -AddressPrefix $vnetsubnet -Subnet $frontendSubnet,$backendsubnet -DnsServer $staticIP1,8.8.8.8
$rule1 = New-AzureRmNetworkSecurityRuleConfig -Name rdp-rule -Description 'Allow RDP' -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389
$rule2 = New-AzureRmNetworkSecurityRuleConfig -Name remoteps-rule -Description 'Allow remote powershell' -Access Allow -Protocol Tcp -Direction Inbound -Priority 101 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 5986
$rule3 = New-AzureRmNetworkSecurityRuleConfig -Name winrm-rule -Description 'Allow winrm' -Access Allow -Protocol Tcp -Direction Inbound -Priority 102 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 5985
$rule4 = New-AzureRmNetworkSecurityRuleConfig -Name SMTP-rule -Description 'SMTP' -Access Allow -Protocol Tcp -Direction Inbound -Priority 103 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 25
$rule5 = New-AzureRmNetworkSecurityRuleConfig -Name HTTP-rule -Description 'HTTP' -Access Allow -Protocol Tcp -Direction Inbound -Priority 104 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 80
$rule6 = New-AzureRmNetworkSecurityRuleConfig -Name HTTPS-rule -Description 'HTTPS' -Access Allow -Protocol Tcp -Direction Inbound -Priority 105 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 443
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $locName -Name $NSGName -SecurityRules $rule1,$rule2,$rule3,$rule4,$rule5,$rule6
Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $frontendsubnetname -AddressPrefix $frontendsubnetrange -NetworkSecurityGroup $nsg | Out-Null
Set-AzureRmVirtualNetwork -VirtualNetwork $vnet | Out-Null
$vnet = Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
write-output '8/14 - Virtual netwrok has been created successfully with two subnets frontend & backend. Also NSG has been created successfully'

########################################################################################################## 
<# 
.SYNOPSIS 
    Create Lab resource group, storage account, virtual network and VMs 
     
.DESCRIPTION 
    This script will create the following components 
    -    Resource group: it will contain all VMs, storage account, virtual network and other resources required for the lab. 
    -    you can edit on labPrefix, labnumber, labsubnet & Ip address with name as you want 
 
.NOTES 
    THIS CODE-SAMPLE IS PROVIDED 'AS IS' WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED  
    OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF MERCHANTABILITY AND/OR  
    FITNESS FOR A PARTICULAR PURPOSE. 
 
 
#> 
########################################################################################################## 
###############Install Azure Module######################### 
Install-Module -Name AzureRM -force 
Import-Module -Name AzureRM 
Install-Module -Name AzureRM.compute 
Import-Module -Name AzureRM.compute 
 
# Connect to Azure 
 
Login-AzurermAccount 
 
# Select Azure Subscription 
$sub = Get-AzureRMsubscription 
Select-AzureRmSubscription -SubscriptionId $sub[0].SubscriptionId 
 
# Set values for new resource group, storage account and vnet 
$labPrefix = 'Mlab' 
$labnumber = '2017' 
$labsubnet = '55' 
$rgName = $labPrefix + $labnumber #New resource group name 
$locName = 'West Europe' # Loation of new resource group 
$saName = $rgName.Replace('-','').tolower() # Storage account name (for new resource group) 
$saType='Standard_LRS' # Storage account type 
$vnetname = $rgName # New virtual network name 
$subnetIndex=0 # Frontend subnet ID 
$frontendsubnetname = $rgName + '-FE' # Frontend subnet name 
$backendsubnetname = $rgName + '-BE' # Backtend subnet name 
$vnetsubnet = '10.$labsubnet.0.0/16' # Virtual network suffix 
$frontendsubnetrange = '10.$labsubnet.1.0/24' # Frontend subnet range 
$backendsubnetrange = '10.$labsubnet.2.0/24' # Backtend subnet name 
$NSGName = $rgName + '-NSG' 
 
## Create initial resources 
# create resource group 
New-AzureRmResourceGroup -Name $rgName -Location $locName | Out-Null 
write-output '1/14 - the resource group has been created successfully' 
# create storage account 
New-AzureRmStorageAccount -Name $saName -ResourceGroupName $rgName –Type $saType -Location $locName | Out-Null 
write-output '2/14 - the storage account has been created successfully' 
# get storage account key 
$key1 = (Get-AzureRmStorageAccountKey -Name $saName -ResourceGroupName $rgName).Value[0] 
# create storage context 
$storagecontext = New-AzureStorageContext -StorageAccountName $saname -StorageAccountKey $key1 
# create a container called scripts 
New-AzureStorageContainer -Name 'scripts' -Context $storagecontext -Permission BLOB  | Out-Null 
write-output '3/14 - Scripts Container has been created successfully' 
New-AzureStorageContainer -Name 'baseimage' -Context $storagecontext -Permission BLOB | Out-Null 
write-output '4/14 - Base Image Container has been created successfully' 
 
 
########################################################## Virtual network config ############################################################## 
 
$frontendSubnet=New-AzureRmVirtualNetworkSubnetConfig -Name $frontendsubnetname -AddressPrefix $frontendsubnetrange 
$backendSubnet=New-AzureRmVirtualNetworkSubnetConfig -Name $backendsubnetname -AddressPrefix $backendsubnetrange 
$vnet = New-AzureRmVirtualNetwork -Name $vnetname -ResourceGroupName $rgName -Location $locName -AddressPrefix $vnetsubnet -Subnet $frontendSubnet,$backendsubnet -DnsServer $staticIP1,8.8.8.8 
$rule1 = New-AzureRmNetworkSecurityRuleConfig -Name rdp-rule -Description 'Allow RDP' -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389 
$rule2 = New-AzureRmNetworkSecurityRuleConfig -Name remoteps-rule -Description 'Allow remote powershell' -Access Allow -Protocol Tcp -Direction Inbound -Priority 101 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 5986 
$rule3 = New-AzureRmNetworkSecurityRuleConfig -Name winrm-rule -Description 'Allow winrm' -Access Allow -Protocol Tcp -Direction Inbound -Priority 102 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 5985 
$rule4 = New-AzureRmNetworkSecurityRuleConfig -Name SMTP-rule -Description 'SMTP' -Access Allow -Protocol Tcp -Direction Inbound -Priority 103 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 25 
$rule5 = New-AzureRmNetworkSecurityRuleConfig -Name HTTP-rule -Description 'HTTP' -Access Allow -Protocol Tcp -Direction Inbound -Priority 104 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 80 
$rule6 = New-AzureRmNetworkSecurityRuleConfig -Name HTTPS-rule -Description 'HTTPS' -Access Allow -Protocol Tcp -Direction Inbound -Priority 105 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 443 
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $locName -Name $NSGName -SecurityRules $rule1,$rule2,$rule3,$rule4,$rule5,$rule6 
Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $frontendsubnetname -AddressPrefix $frontendsubnetrange -NetworkSecurityGroup $nsg | Out-Null 
Set-AzureRmVirtualNetwork -VirtualNetwork $vnet | Out-Null 
$vnet = Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName 
write-output '8/14 - Virtual netwrok has been created successfully with two subnets frontend & backend. Also NSG has been created successfully'




 


You can use this script in following steps:    



  *  Download the script and copy it to your Server or Client machine.

  *  Run Windows PowerShell as Administrator 
  *  .\CreateResourceGroup.ps1. 
**Note:**


  *  You will need to install Azure PowerShell before running this script.

  *  You can edit on labPrefix, labnumber, labsubnet & Ip address with name as you want


    
TechNet gallery is retiring! This script was migrated from TechNet script center to GitHub by Microsoft Azure Automation product group. All the Script Center fields like Rating, RatingCount and DownloadCount have been carried over to Github as-is for the migrated scripts only. Note : The Script Center fields will not be applicable for the new repositories created in Github & hence those fields will not show up for new Github repositories.

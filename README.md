# Powershell-labs <!-- omit in toc -->

Abrir Powershell como administrador

# 1. Intro
```sh
Get-Help
Get-Process
```
## 1.1. Crear una carpeta y archivos
```sh
cd "C:\Users\<USER>"
ls
New-Item -ItemType "Directory" -Name "Temp01"
New-Item -ItemType "File" -Name "File-A" -Path "Temp01"

ls Temp01
```

## 1.2. Permisos
Crear un usuario
```sh
$usuario = "usuario_prueba"
$contraseña = ConvertTo-SecureString "P@ssword123" -AsPlainText -Force

# Crear el usuario local
New-LocalUser -Name $usuario -Password $contraseña -FullName "Usuario de Prueba" -Description "Cuenta temporal para laboratorio"

# Agregarlo al grupo de Usuarios
Add-LocalGroupMember -Group "Users" -Member $usuario

# Verifica
Get-LocalUser -Name $usuario
Get-LocalGroupMember -Group "Users"

```

## 1.3. Agregar permisos
```sh
$carpeta = "C:\Users\<USER>\LaboratorioPermisos"
New-Item -ItemType Directory -Path $carpeta

Get-Acl $carpeta | Format-List

$acl = Get-Acl $carpeta
$permiso = "usuario_prueba","Read","Allow"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule $permiso
$acl.SetAccessRule($rule)
Set-Acl -Path $carpeta -AclObject $acl

# validar
(Get-Acl $carpeta).Access
```

## 1.4. Eliminar permisos
```sh
$acl = Get-Acl $carpeta
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("usuario_prueba","Read","Allow")
$acl.RemoveAccessRule($rule)
Set-Acl -Path $carpeta -AclObject $acl

# validar
(Get-Acl $carpeta).Access

# Eliminar la carpeta
Remove-Item -Path $carpeta -Recurse -Force
```

## 1.5. Eliminar usuarios
```sh
Remove-LocalUser -Name $

# Validar
Get-LocalUser
```

## 1.6. Agregar un alias
```sh
Get-Alias

Set-Alias limpiarTemp "Remove-Item"

limpiarTemp "C:\Users\<USER>\Temp01\*" -Recurse -Force

Get-Alias limpiarTemp
```

# 2. Azure CLI
## 2.1. Login a Azure
```
user1@Storylabscr.onmicrosoft.com/Guca088717
user2@Storylabscr.onmicrosoft.com/Joga440640
user3@Storylabscr.onmicrosoft.com/Buqa595281
user4@Storylabscr.onmicrosoft.com/Suqa062596
```

Ingresar a Azure CLI - Powershell

## 2.2. Setup
```sh
az account list --output table
az account set --subscription "Nombre o ID de la suscripción"
az account show --output table
```


## 2.3. Crear Resource Group
```sh
$rg="rg-NOMBRE"
az group create  --name  $rg  --location eastus
```
> Revisar en Azure el Resource Group

## 2.4. Crear una Máquina Virtual
```sh
$vm="vm-NOMBRE"
$image="Canonical:0001-com-ubuntu-minimal-jammy:minimal-22_04-lts-gen2:latest"
$size = "Standard_B1s"

## Listar imágenes
az vm image list --publisher Canonical --all --output table

az vm create --resource-group $rg --name $vm --image $image --admin-username azureuser --assign-identity --generate-ssh-keys --public-ip-sku Standard --size $size --output table

## Listar las VM - IPs
az vm list --resource-group $rg --output table
az vm list-ip-addresses --output table
```

> Revisar en Azure el Resource Group

> Revisar las VM

## 2.5. Sintaxis alternativa
```sh
$rg2="rg-ps-NOMBRE"

New-AzResourceGroup -Name $rg2 -Location 'EastUS'

# Listar resource groups
Get-AzResourceGroup

# Crear VM
$vm2="vm-ps-NOMBRE"
$image="Debian11"
$ipName="ip-ps-NOMBRE"
$sshKey="ssh-key-ps-NOMBRE"


New-AzVm -ResourceGroupName $rg2 -Name $vm2 -Location 'East US' -image $image -size $size -OpenPorts 80 -GenerateSshKey -SshKeyName $sshKey -PublicIpAddressName $ipName
# Lista2r las VM - IPs
Get-AzVM
Get-AzPublicIpAddress

# Filtrar nombre y IP
Get-AzPublicIpAddress | Select-Object -Property Name,IpAddress
```
> https://aka.ms/findImagePS


## 2.6. Eliminar la VM
```sh
Remove-AzVM -Name $vm2 -ResourceGroupName $rg2 -Force

# Eliminar el Resource Group
Remove-AzResourceGroup -Name $rg2 -Force
Remove-AzResourceGroup -Name $rg -Force
```

## 2.7. Crear un RG + vnet
```sh
$rg3 = "rg3-NOMBRE"
$location = "eastus2"

# Resource Group
New-AzResourceGroup -Name $rg3 -Location $location

# Vnet
$vmName = "vm-lab-NOMBRE"
$vnetName = "$vmName-vnet"
$subnetName = "$vmName-subnet"

$vnet = New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $rg3 -Location $location `
    -AddressPrefix "10.0.0.0/16" `
    -Subnet @(New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix "10.0.0.0/24")

echo $vnet
# Validar la vnet
Get-AzVirtualNetwork -ResourceGroupName $rg3 -Name $vnetName
Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet
```
>  Validar la vnet en Azure


## 2.8. Crear una IP pública
```sh
$ipName = "$vmName-ip"

$publicIp = New-AzPublicIpAddress -Name $ipName -ResourceGroupName $rg3 -Location $location `
    -AllocationMethod Static -Sku Standard

# Validar la IP pública
echo $publicIp
Get-AzPublicIpAddress -ResourceGroupName $rg3
```
- AllocationMethod: Static (permanente) o Dynamic (temporal)
- SKU: Standard (HA, multi AZ, para IP estática)

## 2.9. Crear un NSG
```sh
$nsgName = "$vmName-nsg"
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $rg3 -Location $location -Name $nsgName

# Validar el NSG
echo $nsg
Get-AzNetworkSecurityGroup -ResourceGroupName $rg3 -Name $nsgName
```
## 2.10. Reglas
En este punto existe el NSG, pero no tiene reglas de seguridad.

```sh
$nsgRule = New-AzNetworkSecurityRuleConfig -Name "Allow-HTTP" -Protocol Tcp -Direction Inbound -Priority 1001 `
    -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 80 -Access Allow

$nsg.SecurityRules += $nsgRule
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

# Validar las reglas
Get-AzNetworkSecurityGroup -ResourceGroupName $rg3 -Name $nsgName | Select-Object -ExpandProperty SecurityRules
```

## 2.11. Crear una NIC
```sh
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet
echo $subnet

$nicName = "$vmName-nic"
$nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName $rg3 -Location $location `
    -SubnetId $subnet.Id -PublicIpAddressId $publicIp.Id -NetworkSecurityGroupId $nsg.Id
# Validar la NIC
echo $nic
Get-AzNetworkInterface -ResourceGroupName $rg3 -Name $nicName
# Validar la IP pública
Get-AzPublicIpAddress -ResourceGroupName $rg3 -Name $ipName
```


## 2.12. Crear VM
```sh
$size = "Standard_B1s"
$username = "azureuser"

# listar imagenes Canonical
Get-AzVMImageSku -Location 'East US' -PublisherName Canonical -Offer "ubuntu"
# listar imagenes Canonical Jammy (24)
Get-AzVMImageSku -Location 'East US' -PublisherName Canonical -Offer "0001-com-ubuntu-server-jammy"

# Configuracion
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize $size | `
    Set-AzVMOperatingSystem -Linux -ComputerName $vmName -Credential (Get-Credential -UserName $username -Message "Enter VM credentials") | `
    Set-AzVMSourceImage -PublisherName "Canonical" -Offer "0001-com-ubuntu-server-jammy" -Skus "22_04-lts" -Version "latest" |  `
    Add-AzVMNetworkInterface -Id $nic.Id

# validar vmconfig
echo $vmConfig


# Crear la VM
New-AzVM -ResourceGroupName $rg3 -Location $location -VM $vmConfig

# Validar la VM
$myVM=Get-AzVM -ResourceGroupName $rg3 -Name $vmName
echo $myVM
```


## 2.13. Instalar un webserver
```sh
Invoke-AzVMRunCommand -ResourceGroupName $rg3 -Name $vmName -CommandId 'RunShellScript' -ScriptString 'sudo apt-get update && sudo apt-get install -y nginx'

Get-AzPublicIpAddress | Select-Object -Property Name,IpAddress
```
>  Revisar la IP pública en browser, puerto 80.

# 3. Práctica
- Debe utilizar el resource group 3, la vnet, NSG y la subnet creadas en la sección anterior.

## 3.1. Crear una IP publica
- Nombre: ip-practica-NOMBRE

## 3.2. Crear una NIC
- Nombre nic-practica-NOMBRE
- Asociar la IP pública creada

## 3.3. Listar las imágenes Ubuntu server 20.04 (focal)
Use, offer = 0001-com-ubuntu-server-focal

## 3.4. Crear una VM
- Nombre: vm-practica-NOMBRE
- Imagen: Canonical, 0001-com-ubuntu-server-focal, 20_04-lts
- Tamaño: Standard_B1s
- Nombre usuario: azureuser
- Asociar la NIC creada

## 3.5. Listar la IP pública

## 3.6. Instalar Docker y ejecutar el contenedor: cachac/kubelabs en el puerto 8080
```sh
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo docker run -d -p 8080:8080 cachac/kubelabs:3.0
```
> -p 8080 establece la comunicación por puerto 8080.


## 3.7. Validar en el browser la IP pública, puerto 8080
> El tráfico a puerto 8080 está bloquedo por el NSG.

## 3.8. Crear una regla en el NSG para permitir el tráfico al puerto 8080

## 3.9. Checkpoint
> Revisar el laboratorio antes de continuar.

## 3.10. Eliminar el resource group



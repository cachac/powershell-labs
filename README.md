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

## Crear una Máquina Virtual
```sh
$vm="vm-NOMBRE"
$image="Canonical:0001-com-ubuntu-minimal-jammy:minimal-22_04-lts-gen2:latest"
$size = "Standard_B1s"

# Listar imágenes
az vm image list --publisher Canonical --all --output table

az vm create --resource-group $rg --name $vm --image $image --admin-username azureuser --assign-identity --generate-ssh-keys --public-ip-sku Standard --size $size --output table

# Listar las VM - IPs
az vm list --resource-group $rg --output table
az vm list-ip-addresses --output table
```

> Revisar en Azure el Resource Group

> Revisar las VM

# Comandos en Powershell
```sh
$rg2="rg-ps-NOMBRE"

New-AzResourceGroup -Name $rg2 -Location 'EastUS'

# Listar resource groups
Get-AzResourceGroup

# Crear VM
$vm2="vm-ps-NOMBRE"
$image="Debian11"
New-AzVm -ResourceGroupName $rg2 -Name $vm2 -Location 'East US' -image $image -size $size -OpenPorts 80 -GenerateSshKey -SshKeyName mySSHKey

# Listar las VM - IPs
Get-AzVM
Get-AzPublicIpAddress

# Filtrar nombre y IP
Get-AzPublicIpAddress | Select-Object -Property Name,IpAddress
```
> https://aka.ms/findImagePS


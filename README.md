# Powershell-labs <!-- omit in toc -->

Abrir Powershell como administrador

# Intro
```sh
Get-Help
Get-Process
```
## 2. Crear una carpeta y archivos
```sh
cd "C:\Users\<USER>"
ls
New-Item -ItemType "Directory" -Name "Temp01"
New-Item -ItemType "File" -Name "File-A" -Path "Temp01"

ls Temp01
```

## Permisos
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

## Agregar permisos
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

## Eliminar permisos
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

## Eliminar usuarios
```sh
Remove-LocalUser -Name $

# Validar
Get-LocalUser
```

## 2.1. Agregar un alias
```sh
Get-Alias

Set-Alias limpiarTemp "Remove-Item"

limpiarTemp "C:\Users\<USER>\Temp01\*" -Recurse -Force

Get-Alias limpiarTemp
```

# AZ cli
## Instalar
```sh
winget install --exact --id Microsoft.AzureCLI
az login



Install-Module -Name Az -AllowClobber -Force -SkipPublisherCheck
Import-Module Az

```

# Powershell-labs <!-- omit in toc -->

# 1. Ayuda
```
Get-Help
Get-Process
```
# 2. Crear una carpeta y archivos
```
mkdir temp
touch temp/files
ls temp
```

## 2.1. Agregar un alias
```
Get-Alias

Set-Alias limpiarTemp "Remove-Item"

limpiarTemp "C:\Users\<USER>\AppData\Local\Temp\*" -Recurse -Force

Get-Alias limpiarTemp
```

Storylabscr2025#
SRS7GWY8Svi63G

```sh
# comando para instalar Docker y ejecutar el contenedor
Invoke-AzVMRunCommand -ResourceGroupName $rg3 -Name <NOMBRE> -CommandId 'RunShellScript' -ScriptString 'sudo apt-get update && sudo apt-get install -y docker.io && sudo systemctl start docker && sudo systemctl enable docker && sudo docker run -d -p 8080:80 cachac/kubelabs:3.0'

# regla para permitir el tráfico al puerto 8080
$nsgRule = New-AzNetworkSecurityRuleConfig -Name "Allow-HTTP-8080" -Protocol Tcp -Direction Inbound -Priority 1002 `
	-SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 8080 -Access Allow

# agregar la regla al NSG
$nsg.SecurityRules += $nsgRule
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg
```

---
description: Mostly borrowed...
---

# WinRM

Research:

WinRM Research - this is closest to my lab [https://github.com/jmassardo/Azure-WinRM-Terraform](https://github.com/jmassardo/Azure-WinRM-Terraform) WinRM in a domain -&gt; [http://www.anniehedgie.com/terraform-and-winrm](http://www.anniehedgie.com/terraform-and-winrm)   
Sample tested on W2012R2 [https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-windows)   
Later version with W2019 [https://www.starwindsoftware.com/blog/azure-execute-commands-in-a-vm-through-terraform](https://www.starwindsoftware.com/blog/azure-execute-commands-in-a-vm-through-terraform)   
NEED TO TEST IF WINRM IS LISTENING? [https://stevenmurawski.com/2015/06/need-to-test-if-winrm-is-listening/](https://stevenmurawski.com/2015/06/need-to-test-if-winrm-is-listening/) [https://registry.terraform.io/modules/claranet/windows-vm/azurerm/latest](https://registry.terraform.io/modules/claranet/windows-vm/azurerm/latest)

Creating the listener and configuring Autologon

```text
  winrm_listener {
    protocol = "Http"
  }

  copied from https://github.com/claranet/terraform-azurerm-windows-vm

  secret {
    key_vault_id = azurerm_key_vault.tabwinkv.id
    #key_vault_id = var.key_vault_id
    certificate {
      url   = azurerm_key_vault_certificate.winrm_certificate.secret_id
      store = "My"
    }
  }
}
  # Auto-Login's required to configure WinRM
  additional_unattend_content {
    setting = "AutoLogon"
    content = "<AutoLogon><Password><Value>${var.admin_password}</Value></Password><Enabled>true</Enabled><LogonCount>1</LogonCount><Username>${var.admin_username}</Username></AutoLogon>"
  }

  # Unattend config is to enable basic auth in WinRM, required for the provisioner stage.
  additional_unattend_content {
    setting = "FirstLogonCommands"
    content = file("./files/FirstLogonCommands.xml") #file(format("%s/files/FirstLogonCommands.xml", path.module))
  }
  # https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
  identity {
    type = "SystemAssigned"
  }

  # https://docs.microsoft.com/en-us/azure/virtual-machines/custom-data
  # https://github.com/terraform-providers/terraform-provider-azurerm/issues/6138
    custom_data    = base64encode(local.custom_data_content)
}
```



Not sure where I got this from:

```text
$profiles = Get-NetConnectionProfile
Foreach ($i in $profiles) {
    Write-Host ("Updating Interface ID {0} to be Private.." -f $profiles.InterfaceIndex)
    Set-NetConnectionProfile -InterfaceIndex $profiles.InterfaceIndex -NetworkCategory Private
}

Write-Host "Obtaining the Thumbprint of the Certificate from KeyVault"
$Thumbprint = (Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -match "$ComputerName"}).Thumbprint

Write-Host "Enable HTTPS in WinRM.."
winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"$ComputerName`"; CertificateThumbprint=`"$Thumbprint`"}"

Write-Host "Enabling Basic Authentication.."
winrm set winrm/config/service/Auth "@{Basic=`"true`"}"

Write-Host "Re-starting the WinRM Service"
net stop winrm
net start winrm

Write-Host "Open Firewall Ports"
netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=5986
```

Copied from [https://github.com/claranet/terraform-azurerm-windows-vm](https://github.com/claranet/terraform-azurerm-windows-vm)

```text
resource "null_resource" "winrm_connection_test" {
  count = var.public_ip_sku == null ? 0 : 1

  depends_on = [
    azurerm_network_interface.nic,
    azurerm_public_ip.publicip,
    azurerm_windows_virtual_machine.windows_vm,
  ]

  # https://www.cloudmanav.com/terraform/executing-scripts-terraform-template/#  
  triggers = {
    uuid = azurerm_windows_virtual_machine.windows_vm.id
  }

  connection {
    type     = "winrm"
    host     = join("", azurerm_public_ip.publicip.*.ip_address)
    port     = 5986
    https    = true
    user     = var.admin_username
    password = var.admin_password
    timeout  = "3m"

    # NOTE: if you're using a real certificate, rather than a self-signed one, you'll want this set to `false`/to remove this.
    insecure = true
  }

  # https://www.terraform.io/docs/language/resources/provisioners/file.html
  provisioner "file" {
    source      = "files/New-ScheduledTask.ps1"
    destination = "C:\\jt365\\New-ScheduledTask.ps1"
  }

  provisioner "file" {
    source      = "files/wintab-deploy-original.ps1"
    destination = "C:\\jt365\\wintab-deploy-original.ps1"
  }

  provisioner "remote-exec" {
    inline = [
      "cd C:\\jt365",
      "dir",
      "PowerShell.exe -ExecutionPolicy Bypass -File C:\\jt365\\New-ScheduledTask.ps1",
    ]
  }
}
```


---
description: Comparing the approaches and code for deploying Tableau into Azure.
---

# WinRM Provisioner Connection or CustomScriptExtension

## The Challenge... 

...if you wish to accept:

**Complete an unattended install of Tableau on a Windows Server in Azure.**

There were two approaches that I found to configure the Windows server post deployment. Using WinRm or the CustomScriptExtension in Azure. I wish I had found the CustomScriptExtension sooner...

## WinRM Approach

Firstly, a big thank you goes out to Claranet as they have contributed a whole host of useful modules to the Terraform registry. As I am learning Terraform I wanted to beg, borrow and steal to _build_ my own, and [this module](https://registry.terraform.io/modules/claranet/windows-vm/azurerm/latest) was a great starting point. I _borrowed_ not only the code but also the structure of the module as part of my Terraform learning experience. 

The high-level process is:

1. Create a Key Vault and self-signed certificate
2. Deploy the server
3. Install a self-signed certificate on the server
4. Configure the server to automatically logon
5. Copy a file that will enable and configure WinRM
6. Run that file
7. Test the WinRM connection
8. Copy the Tableau installation script
9. Run the install script  

Simples.

### Fun with Key Vault

The key vault was the area where I got the most issues. This showed up my 'Just do it' approach to coding. My way of working with Terraform has been more try it and work back from the problem. That takes me on some nice detours but Key Vaults require some thought and design.

I had problems with deleting the certificate, problems with recreating the certificate, problems with naming, deleting the vault, problems with permissions you name it. 

The code below kind of works but I usually get an error reported in terraform cli about deleting the certificate, however when you check the Azure Portal it has worked. Lost patience in trying to find out why. 

```text
resource "azurerm_key_vault" "tabwinkv" {
  name                        = "tabwinkv-terrazure"
  location                    = azurerm_resource_group.rg.location
  resource_group_name         = azurerm_resource_group.rg.name
  enabled_for_disk_encryption = true
  tenant_id                   = var.tenant_id
  soft_delete_retention_days  = 7
  purge_protection_enabled    = false
  enabled_for_deployment      = true
  sku_name = "standard"

  access_policy {
    tenant_id = var.tenant_id
    object_id = data.azurerm_client_config.current.object_id

    certificate_permissions = [
      "create",
      "delete",
      "deleteissuers",
      "get",
      "getissuers",
      "import",
      "list",
      "listissuers",
      "managecontacts",
      "manageissuers",
      "setissuers",
      "update",
      "recover",
      "backup",
      "restore",
      "purge"
    ]

    key_permissions = [
      "backup",
      "create",
      "decrypt",
      "delete",
      "encrypt",
      "get",
      "import",
      "list",
      "purge",
      "recover",
      "restore",
      "sign",
      "unwrapKey",
      "update",
      "verify",
      "wrapKey",
    ]

    secret_permissions = [
      "backup",
      "delete",
      "get",
      "list",
      "purge",
      "recover",
      "restore",
      "set",
    ]
  }
}

resource "azurerm_key_vault_certificate" "winrm_certificate" {
  name         = "winrm-${var.prefix}-cert"
  key_vault_id = azurerm_key_vault.tabwinkv.id
  # key_vault_id = var.key_vault_id
  certificate_policy {
    issuer_parameters {
      name = "Self"
    }

    key_properties {
      exportable = true
      key_size   = 2048
      key_type   = "RSA"
      reuse_key  = true
    }

    lifetime_action {
      action {
        action_type = "AutoRenew"
      }

      trigger {
        days_before_expiry = 30
      }
    }

    secret_properties {
      content_type = "application/x-pkcs12"
    }

    x509_certificate_properties {
      # Server Authentication = 1.3.6.1.5.5.7.3.1
      # Client Authentication = 1.3.6.1.5.5.7.3.2
      extended_key_usage = ["1.3.6.1.5.5.7.3.1"]

      key_usage = [
        "cRLSign",
        "dataEncipherment",
        "digitalSignature",
        "keyAgreement",
        "keyCertSign",
        "keyEncipherment",
      ]

      subject            = "CN=${var.prefix}-TFVM"
      validity_in_months = var.certificate_validity_in_months
    }
  }
}
```

### Windows VM deployment

As part of the Windows-VM module there are blocks to create the listener and configure Autologon.

```text
  # Installing a certificate from KeyVault into the local certificate store
  secret {
    key_vault_id = azurerm_key_vault.tabwinkv.id
    #key_vault_id = var.key_vault_id
    certificate {
      url   = azurerm_key_vault_certificate.winrm_certificate.secret_id
      store = "My"
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
}
```

[FirstLogonCommands.xml ](https://docs.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-firstlogoncommands)is used to create the directory to store scripts. It also copies the winrm.ps1 configuration script and runs it with the correct PowerShell parameters.

```text
<FirstLogonCommands>
    <SynchronousCommand>
        <CommandLine>cmd /c "mkdir C:\jt365"</CommandLine>
        <Description>Create the Terraform working directory</Description>
        <Order>11</Order>
    </SynchronousCommand>
    <SynchronousCommand>
        <CommandLine>cmd /c "copy C:\AzureData\CustomData.bin C:\jt365\winrm.ps1"</CommandLine>
        <Description>Move the CustomData file to the working directory</Description>
        <Order>12</Order>
    </SynchronousCommand>
    <SynchronousCommand>
        <CommandLine>powershell.exe -sta -ExecutionPolicy Unrestricted -file C:\jt365\winrm.ps1</CommandLine>
        <Description>Execute the WinRM enabling script</Description>
        <Order>13</Order>
    </SynchronousCommand>
</FirstLogonCommands>
```

100% borrowed. Not sure where I got this from, but thanks.

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

### WinRM Connection

This creates a remote [connection](https://www.terraform.io/docs/language/resources/provisioners/connection.html) to the server using WinRM to copy and execute PowerShell scripts. 

I had inadvertently commented out `insecure = true` and was getting this error Warning: `"skip_credentials_validation": [DEPRECATED] This field is deprecated and will be removed in version 3.0 of the Azure Provider` 

```text
resource "null_resource" "winrm_connection_test" {
  count = var.public_ip_sku == null ? 0 : 1

  depends_on = [
    azurerm_network_interface.nic,
    azurerm_public_ip.publicip,
    azurerm_windows_virtual_machine.windows_vm,
  ]

#   # https://www.cloudmanav.com/terraform/executing-scripts-terraform-template/#  
  triggers = {
    uuid = azurerm_windows_virtual_machine.windows_vm.id
  }


# NOTE: if you're using a real certificate, rather than a self-signed one, you'll want this set to `false`/to remove this.
  connection {
    type     = "winrm"
    host     = join("", azurerm_public_ip.publicip.*.ip_address)
    port     = 5986
    https    = true
    user     = var.admin_username
    password = var.admin_password
    timeout  = "40m"
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
      "PowerShell.exe -ExecutionPolicy Bypass -File C:\\jt365\\wintab-deploy-original.ps1"
    ]
  }
}
```

### 2 main issues

Initial issue was with [Start-Bitstransfer](https://docs.microsoft.com/en-us/windows/win32/bits/using-windows-powershell-to-create-bits-transfer-jobs) only working when a user is logged on.

> When you use [\*-BitsTransfer](https://docs.microsoft.com/en-us/previous-versions//dd819413%28v=technet.10%29) cmdlets from within a process that runs in a noninteractive context, such as a Windows service, you may not be able to add files to BITS jobs, which can result in a suspended state. For the job to proceed, the identity that was used to create a transfer job must be logged on. For example, when creating a BITS job in a PowerShell script that was executed as a Task Scheduler job, the BITS transfer will never complete unless the Task Scheduler's task setting "Run only when user is logged on" is enabled.

Changed that to .Net method to download file. Thank you [AdamTheAutomator](https://adamtheautomator.com/powershell-download-file/) 

`[System.Net.WebClient]::new().DownloadFile($url,$($folder+$DownloadFile))`

After doing these changes I found that Timeouts were occurring:

```text
null_resource.winrm_connection_test[0]: Still creating... [8m40s elapsed]
null_resource.winrm_connection_test[0]: Still creating... [8m50s elapsed]
null_resource.winrm_connection_test[0]: Still creating... [9m0s elapsed]
null_resource.winrm_connection_test[0]: Still creating... [9m10s elapsed]
null_resource.winrm_connection_test[0]: Still creating... [9m20s elapsed]


Error: error executing "C:/Temp/terraform_1958524305.cmd": unknown error Post "https://13.93.203.89:5986/wsman": net/http: timeout awaiting response headers
```

Fixed by extending the WinRM timeout - `winrm set winrm/config @{MaxTimeoutms="27000000"}`

I also got an Error setting PATH in the script... but that is for another day.

```text
[2021/03/20/ 16:39:33], Adding TSM to local Windows system PATH
null_resource.winrm_connection_test[0] (remote-exec): Get-ItemProperty : Cannot find path 'C:\Program Files\Tableau\Tableau Server\Packages' because it does not exist.
```

### Useful Stuff

WinRM Research - [https://github.com/jmassardo/Azure-WinRM-Terraform](https://github.com/jmassardo/Azure-WinRM-Terraform)   
WinRM in a domain -&gt; [http://www.anniehedgie.com/terraform-and-winrm](http://www.anniehedgie.com/terraform-and-winrm)   
W2019 -&gt; [https://www.starwindsoftware.com/blog/azure-execute-commands-in-a-vm-through-terraform](https://www.starwindsoftware.com/blog/azure-execute-commands-in-a-vm-through-terraform)   
How to test if WinRM is listening -&gt; [https://stevenmurawski.com/2015/06/need-to-test-if-winrm-is-listening/](https://stevenmurawski.com/2015/06/need-to-test-if-winrm-is-listening/)

## The CustomScriptExtension approach

The high-level process is:

1. Install the CustomScriptExtension
2. Run the Tableau install script

Yes. That's it.

Firstly make sure you follow the guidance on the [Terraform docs](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_machine_extension):

> The `Publisher` and `Type` of Virtual Machine Extensions can be found using the Azure CLI

I played around too much without following this guidance and wasted time. You should go straight to the [Azure docs](https://docs.microsoft.com/en-us/cli/azure/vm/extension?view=azure-cli-latest) and run the `image list`, `list-versions` and `image show` examples.

One of the challenges I found was that it would only work on minor versions, not a patch e.g. 1.10 not 1.10.5 - Maybe I need to use the minor version upgrade thingy?

Keep on getting this error after 30 minutes but the installtakes longer, around 40 minutes! Error: Future\#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded

```text
timeouts {
     create = "60m"
     delete = "2h"
   }
```




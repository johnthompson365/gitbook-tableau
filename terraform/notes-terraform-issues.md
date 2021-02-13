---
description: Tracker
---

# Terraform Issues

I'm not saying key vaults are tricky but....

<table>
  <thead>
    <tr>
      <th style="text-align:left">Issue</th>
      <th style="text-align:left">Error</th>
      <th style="text-align:left">Resolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Azure Key Vault Certificate</td>
      <td style="text-align:left">
        <p>Error: Code=&quot;VMExtensionProvisioningError&quot; Message=&quot;VM
          has reported a failure when processing extension &apos;TABWIN-TFVM-keyvaultextension&apos;.
          Error message: \&quot;observedCertificates cannot be empty\&quot;\r\n\r\nMore
          information on troubleshooting is available at <a href="https://aka.ms/vmextensionwindowstroubleshoot">https://aka.ms/vmextensionwindowstroubleshoot</a> &quot;</p>
        <p>on certificate.tf line 62, in resource &quot;azurerm_virtual_machine_extension&quot;
          &quot;keyvault_certificates&quot;: 62: resource &quot;azurerm_virtual_machine_extension&quot;
          &quot;keyvault_certificates&quot; {</p>
      </td>
      <td style="text-align:left"><b>In progress</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Azure Key Vault Certificate</td>
      <td style="text-align:left">Error: purging Certificate &quot;winrm-TABWIN-cert&quot; keyvault.BaseClient#PurgeDeletedCertificate:
        Failure responding to request: StatusCode=403 -- Original Error: autorest/azure:
        Service returned an error. Status=403 Code=&quot;Forbidden&quot; Message=&quot;The
        user, group or application &apos;appid=04b07795-&apos;xxx does not have
        certificates purge permission on key vault &apos;tabwin&apos;</td>
      <td style="text-align:left">
        <p><b>To be tested</b>
        </p>
        <p>certificate_permissions = [&quot;purge&quot; ]</p>
        <p></p>
        <p>didn&apos;t work I am assuming it is something to do with <a href="https://stackoverflow.com/questions/61342357/unable-to-delete-secrets-from-key-vault-with-soft-delete-enabled">Purge Protection</a>
        </p>
        <p></p>
        <p>Again No, as Purge Protection is not enabled. Only soft-delete.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Variables</td>
      <td style="text-align:left">
        <p>Warning: Value for undeclared variable</p>
        <p>The root module does not declare a variable named &quot;tenant_id&quot;
          but a value was found in file &quot;terraform.tfvars&quot;. To use this
          value, add a &quot;variable&quot; block to the configuration.</p>
      </td>
      <td style="text-align:left">
        <p>Now that is a useful error message.</p>
        <p>If you want to use terraform.tfvars, the variable also has to be declared
          in the variables.tf</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">Error: purging Certificate &quot;winrm-ADFS-cert&quot; (Key Vault &quot;
        <a
        href="https://tabwinkv.vault.azure.net/">https://tabwin/</a>&quot;): keyvault.BaseClient#PurgeDeletedCertificate:
          Failure responding to request: StatusCode=403 -- Original Error: autorest/azure:
          Service returned an error. Status=403 Code=&quot;Forbidden&quot; Message=&quot;The
          user, group or application &apos;appid=04 does not have certificates purge
          permission on key vault &apos;tabwinkv;location=westus&apos;. InnerError={&quot;code&quot;:&quot;ForbiddenByPolicy&quot;}</td>
      <td
      style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Vault again</td>
      <td style="text-align:left">Message=&quot;The vault name &apos;xxx&apos; is already in use. Vault
        names are globaly unique so it is possible that the name is already taken.
        If you are sure that the vault name was not taken then it is possible that
        a vault with the same name was recently deleted but not purged after being
        placed in a recoverable state. If the vault is in a recoverable state then
        the vault will need to be purged before reusing the name.</td>
      <td style="text-align:left">
        <p>I had not deleted the vault in my other subscription. There need to be
          some creative vault names if they are globally unique!</p>
        <p></p>
        <p>I changed the name and it worked. Nice.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>Error checking for presence of existing Certificate</p>
        <p>keyvault.BaseClient#GetCertificate: Failure responding to request: StatusCode=403</p>
        <p>does not have certificates get permission on key vault</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">Error: Cycle: azurerm_key_vault_access_policy.vm, azurerm_key_vault_certificate.winrm_certificate,
        azurerm_windows_virtual_machine.windows_vm</td>
      <td style="text-align:left"><a href="https://serverfault.com/questions/1005761/what-does-error-cycle-means-in-terraform">https://serverfault.com/questions/1005761/what-does-error-cycle-means-in-terraform</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">Error: Error ensuring Resource Providers are registered.</td>
      <td style="text-align:left"><a href="https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_provider_registration">https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_provider_registration</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">Error: waiting for creation of Windows Virtual Machine &quot;TABWIN-TFVM&quot;
        (Resource Group &quot;TABWIN-TFrg&quot;): Code=&quot;KeyVaultAccessForbidden&quot;
        Message=&quot;Key Vault <a href="https://tabwinkv-jt365tb01.vault.azure.net/secrets/winrm-TABWIN-cert/58a87f1a91284a49a0bd6600f80a21b9">https://tabwinkv-jt365tb01.vault.azure.net/secrets/winrm-TABWIN-</a>either
        has not been enabled for deployment or the vault id provided, /subscriptions/x/resourceGroups/TABWIN-TFrg/providers/Microsoft.KeyVault/vaults/tabwinkv-jt365tb01,
        does not match the Key Vault&apos;s true resource id.&quot;</td>
      <td style="text-align:left">enabled_for_deployment = true</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">Error: A resource with the ID &quot;/subscriptions/x/resourceGroups/TABWIN-TFrg/providers/Microsoft.Compute/virtualMachines/TABWIN-TFVM&quot;
        already exists - to be managed via Terraform this resource needs to be
        imported into the State. Please see the resource documentation for &quot;azurerm_windows_virtual_machine&quot;
        for more information.</td>
      <td style="text-align:left">terraform destroy</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">already exists - to be managed via Terraform this resource needs to be
        imported into the State</td>
      <td style="text-align:left">
        <p>terraform import azurerm_windows_virtual_machine.windows_vm /subscriptions/x/resourceGroups/TABWIN-TFrg/providers/Microsoft.Compute/virtualMachines/TABWIN-TFVM</p>
        <p></p>
        <p>terraform destroy</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">Error: creating Windows Virtual Machine &quot;TABWIN-TFVM&quot; (Resource
        Group &quot;TABWIN-TFrg&quot;): compute.VirtualMachinesClient#CreateOrUpdate:
        Failure sending request: StatusCode=0 -- Original Error: autorest/azure:
        Service returned an error. Status= Code=&quot;ConflictingUserInput&quot;
        Message=&quot;Disk TABWIN-OsDisk already exists in resource group TABWIN-TFRG.
        Only CreateOption.Attach is supported.&quot; Target=&quot;/subscriptions/x/resourceGroups/TABWIN-TFrg/providers/Microsoft.Compute/disks/TABWIN-OsDisk&quot;</td>
      <td
      style="text-align:left"></td>
    </tr>
  </tbody>
</table>


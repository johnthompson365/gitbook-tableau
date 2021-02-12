---
description: Tracker
---

# Terraform Issues

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
  </tbody>
</table>


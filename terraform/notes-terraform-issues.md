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
  </tbody>
</table>


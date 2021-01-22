# Notes: Terraform Issues

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
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>


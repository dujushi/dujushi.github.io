Powershell makes generating a new self-signed certificate quite easy. With one simple command, you can have one installed on your machine. A PFX file can be exported for other usages. 

## Creates a new self-signed certificate
You need to run PowerShell as an administrator, otherwise it will throw an "Access denied" error. 

{% highlight powershell %}
$certificateDnsName = "your.dns.name"
$certificateStoreLocation = "cert:\LocalMachine\My"
New-SelfSignedCertificate -DnsName $certificateDnsName -CertStoreLocation $certificateStoreLocation
{% endhighlight %}

## Generate a PFX file
{% highlight powershell %}
$certificateDnsName = "your.dns.name"
$certificateStoreLocation = "cert:\LocalMachine\My"
$pfxPassword = "yourpasswordgenerator"
$pfxLocation = "C:\your_pfx_name.pfx"

$newSelfSignedCertificateResult = New-SelfSignedCertificate -DnsName $certificateDnsName -CertStoreLocation $certificateStoreLocation
$secureString = ConvertTo-SecureString -String $pfxPassword -Force -AsPlainText
$certificatePath = Join-Path -Path $certificateStoreLocation -ChildPath $newSelfSignedCertificateResult.Thumbprint
Get-ChildItem -Path $certificatePath | Export-PfxCertificate -FilePath $pfxLocation -Password $secureString
{% endhighlight %}

## References
1. [New-SelfSignedCertificate](https://docs.microsoft.com/en-us/powershell/module/pkiclient/new-selfsignedcertificate){:target="_blank"}
2. [Export-PfxCertificate](https://docs.microsoft.com/en-us/powershell/module/pkiclient/export-pfxcertificate){:target="_blank"}
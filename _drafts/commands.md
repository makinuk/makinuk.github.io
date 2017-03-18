//pfx converter
openssl pkcs12 -in celcus.pfx -clcerts -nokeys -out celcus.cer
openssl pkcs12 -in celcus.pfx -nocerts -nodes  -out celcus_encrypted.key
openssl rsa -in celcus_encrypted.key -out celcus.key

##openssl pkcs12 -in domain.pfx -clcerts -nokeys -out domain.cer
##openssl pkcs12 -in domain.pfx -nocerts -nodes  -out domain.key
##openssl pkcs12 -in domain.pfx -out domain-ca.crt -nodes -nokeys -cacerts
##<VirtualHost 192.168.0.1:443>
## ...
## SSLEngine on
## SSLCertificateFile /path/to/domain.cer
## SSLCertificateKeyFile /path/to/domain.key
## SSLCACertificateFile /path/to/domain-ca.crt
## ...
##</VirtualHost>


//csr ve key yaratma
openssl req -nodes -newkey rsa:2048 -keyout myserver.key -out server.csr
openssl req -nodes -newkey rsa:2048 -nodes -keyout myserver.key -out server.csr -subj "/C=TR/ST=Istanbul/L=Istanbul/O=Mtm Medya Takip Merkezi/OU=Celsus/CN=celsus_test.mtm.local"

//Self Signed SSSL Create Process
openssl genrsa -out server.key 2048
openssl req -new -x509 -key server.key -out server.crt -days 1825



//pfx file to key file
openssl pkcs12 -in [yourfile.pfx] -nocerts -out [keyfile-encrypted.key]
//pfx file to crt file
openssl pkcs12 -in [yourfile.pfx] -clcerts -nokeys -out [certificate.crt]
//encrypted key to decrypted key
openssl rsa -in [keyfile-encrypted.key] -out [keyfile-decrypted.key]


//working Process
Get-WmiObject win32_Process | where-object {$_.CommandLine -like "*php*"} | Select-Object ProcessId,CommandLine,ProcessName,CreationDate | Format-List

//kill all php 
(get-process |where {$_.ProcessName -eq "php.exe"}) | stop-process

//working process table list 
Get-WmiObject win32_Process | where-object {$_.CommandLine -like "*php*"} | Select-Object ProcessId,CommandLine,ProcessName,CreationDate | Format-Table -wrap

// kill process working more than 15 min 
(Get-WmiObject win32_Process | where-object {$_.CommandLine -like '*download-file.php*' -and $_.ConvertToDateTime($_.CreationDate) -lt (Get-Date).AddMinutes(-15)}).Invokemethod("terminate", $null)

//Search Local CN (Common Name ) in Local AD (Active Directory)
Get-ADObject -Filter { CN -like "*Mustafa*" }

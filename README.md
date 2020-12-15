# fncm-code-signing-signature-files
## IBM FileNet Content Manager Code Signing Signature, Public Keys, and Public Certificates

This repository contains signature files, public keys, and public certificates required to verify signed binaries for IBM FileNet Content Manager and IBM Content Foundation products. The signature files can be used verify that your downloaded binaries match the files that were built and released by IBM and have not been tampered with or modified. Note that signature files for Windows binaries (such as executables) and for zip files are embedded in those files. The files contained on this repository are only required for validating AIX, Linux, and zLinux binaries, as the signature files for those binaries are separate.

To verify your downloaded binaries, follow the instructions below.

---
### Verifying AIX, Linux, and zLinux Binaries

The signature files for AIX, Linux, and zLinux binaries can be found in this repository under the component and version directory structure. You will also find any files needed to verify the authenticity of the signature files in this repository. Signature files are either given similar names to the original file with a .sig suffix or a name matching the part number of the original file with a .sig suffix. Public keys and public certificates are generally given a .pem suffix. Under each version folder can be one of several folders with additional files:
- docker-image - Signature file for the tgz package containing the Docker image. Note that the image inside of the package is also signed, though the signature for the image is embedded inside of the image itself.
- keys-certificates - Contains the following files that can be used to validate that a signature file is valid:
  - certificate.pem - The public certificate used to generate the signature file
  - intermediate.pem - the public intermediary certificate (outside this repository the certificate might be named chain0.pem)
  - publickey.pem - The public key for validating the signature
- media - Signature files for the installation media package, in which the installers and other files related to the component can be found (usally tar.gz or tgz files)

You can verify the file using the signature and public key files with the OpenSSL command.

```
openssl dgst -sha256 -verify <publickey.pem> -signature <SIGNATUREFILENAME> <FILETOSIGN> 
```

If the signature is valid and matches with the signed file and the public key, the output will be:
```
Verified OK
```
Following is the example how to verify the CPE 5.5.6 Linux image:
```
[root@scow1 tmp]# openssl dgst -sha256 -verify publickey.pem -signature IBM_FILENET_CPE_V5.5.6_LINUX_ML.tar.gz.sig IBM_FILENET_CPE_V5.5.6_LINUX_ML.tar.gz
Verified OK
```

If the signature is not valid, verify that you are comparing the correct signature file and public key to the correct file. Also, you may consider redownloading the binary, signature file, and public key. If the signature is still reported as not valid, you can contact IBM customer support for help.

You can validate that the public key is present in the certificate and the certificate is still valid using the following instructions. 

#### How to compare the certificate and public key 
```
openssl x509 -text -in <certificate.pem> -noout #shows the certificate details, e.g. it is signed by IBM and Digicert
openssl rsa -noout -text -inform PEM -in <publickey.pem> -pubin   #shows the public key details
```
Compare the exponent/data of the public key and the certificate to see that the public key is indeed the one within the certificate. 

Certificate Modulus:
```
                    00:e2:45:27:25:e9:a3:1f:c2:37:27:ac:4c:89:86:
					
                    ae:32:d5:2a:84:69:3b:01:cb:54:34:b0:b3:1b:6d: .......

                     Exponent: 65537 (0x10001)
```
Public key:
```
               00:e2:45:27:25:e9:a3:1f:c2:37:27:ac:4c:89:86:

              ae:32:d5:2a:84:69:3b:01:cb:54:34:b0:b3:1b:6d: .......

                     Exponent: 65537 (0x10001)
```

If the modulus for the certificate and public key do not match, verify that you are comparing the correct certificate and public key. Also, you may consider redownloading the public key and certificate. If the modulus for the certificate and public key still do not match, you can contact IBM customer support for help.

#### How to check the IBM public certificate validity
```
openssl ocsp -no_nonce -issuer <intermediate.pem> -cert <certificate.pem> -VAfile <intermediate.pem> -text -url http://ocsp.digicert.com -respout ocsptest     #check if the cert is still valid
```
If the certificate is valid, the output will be: 
```
Response verify OK
certificate.pem: good
```

If the certificate is valid, but expired, the output will be:
```
Response verify OK
certificate.pem: WARNING: Status times invalid.
140320062572432:error:2707307D:OCSP routines:OCSP_check_validity:status expired:ocsp_cl.c:372:good
```

If the certificate is not valid, verify that you are using the correct public certificate and public intermediate certificate.  Also, verify that you are able to connect to http://ocsp.digicert.com from the system you are running the commands on and that the date and time on your system are correct and match a trusted Network Time Protocol (NTP) server. After changing the time, you may need to restart the server before verifying the certificate. If the certificate is still reported as not valid, you can contact IBM customer support for help.

---
### Verifying Zip Files

To verify a zip file, you must first install a Java SDK on your system (an IBM Java SDK may be found in the WebSphere Application Server installation, but an Oracle Java SDK or an OpenJDK may also be used), use the following command:
```
<JDKPATH>/bin/jarsigner -verify -certs -verbose <SIGNEDZIPFILEPATH>
```

If the signature is valid, the output will be:
```
jar verified.

The signer certificate will expire on 2021-08-05.
The timestamp will expire on 2030-10-16.
```

If the signature is valid, but the signer certificate has expired expired, the output will be:
```
jar verified.

The signer certificate expired on 2021-08-05. However, the JAR will be valid until the timestamp expires on 2030-10-16.
```

If the signature for the zip file is not valid, verify that you are using the correct zip file. Or confirm the keystore for the Java SDK you are using contains a current DigiCert Certificate Authority certificate. If in doubt, try a newer or different Java SDK. If the signature for the zip file is still reported as not valid, you can contact IBM customer support for help.

---
### Verifying Windows Binaries

To verify a Windows binary, such as an executable file, right click on the artifact, click Properties/Digital signatures, and you should see the IBM code signing certificates. You can inspect the IBM code signing certificates to make sure that they are valid and have not expired. If the file has no Digital Signatures Tab under Properties, it means that it is unsigned.

---
### Expiration of signature files, public keys, and certificates

The IBM certificates used to sign a file generally have a lifespan of about three years. If you are concerned about the validity of your artifacts after the certificate used to sign the artifact has expired, you may contact IBM support to request newly generated signature files, public keys, public certificates, or installation media.

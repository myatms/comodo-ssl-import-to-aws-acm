# Install your Comodo Certificates to Amazon AWS

Comodo, the leading Internet Security Provider offers Free Antivirus, SSL Certificate and other Internet Security related products with complete protection. In this post I will walk you through the setup of SSL in Amazon CloudFront (the process is common to all Amazon services)

AWS need that all your certificates are in **PEM format**. They are two main of encoding certificate:

DER: is a binary encoding of a certificate. Typically these use the file extension of .crt or .cert.

PEM: is a Base64 encoding of a certificate represented in ASCII therefore it is readable as a block of text. This is very useful as you can open it in a text editor work with the data more easily.
Comodo certificate are delivered in DER format **.crt**, so we need to **convert them to PEM**.

## Certificates Setup

### Convert crt to PEM

Amazon AWS need:

- Your issued certificate
- Your private key
- The CAChain certificate that include all intermediate and Root CA certificate.

Comodo send you 4 certificates:

- AAACertificateServices.crt
- STAR.example.com.crt
- USERTrustRSAAAACA.crt
- SectigoRSADomainValidationSecureServerCA.crt

First cding to the folder containing all your certificates:

```bash
$ cd /path/to/certificates/folder
$ mkdir pem
```

Then convert all certificates:

```sh
openssl x509 -in ./AAACertificateServices.crt -outform pem -out ./pem/AAACertificateServices.pem
openssl x509 -in ./USERTrustRSAAAACA.crt -outform pem -out ./pem/USERTrustRSAAAACA.pem
openssl x509 -in ./SectigoRSADomainValidationSecureServerCA.crt -outform pem -out ./pem/SectigoRSADomainValidationSecureServerCA.pem
openssl x509 -in ./STAR.example.com.crt -outform pem -out ./pem/STAR.example.com.pem
```

**x509**: The x509 command is a multi purpose certificate utility. It can be used to display certificate information, convert certificates to various forms, sign certificate requests like a “mini CA” or edit certificate trust settings.
**-in <filename>**: This specifies the input filename to read a certificate from or standard input if this option is not specified.
**-outform PEM**: This specifies the output format. In this case PEM.
**-out filename**: This specifies the output filename to write to or standard output by default.

Convert the private key:

```sh
openssl rsa -in ./private.key -outform PEM -out private.key.pem
```

*rsa*: The rsa command processes RSA keys.

### Create a CAChain

```sh
$ cat ./pem/SectigoRSADomainValidationSecureServerCA.pem > ./pem/CAChain.pem
$ cat ./pem/USERTrustRSAAAACA.pem >> ./pem/CAChain.pem
```

> Warning: You must construct the CAChain in descending order. Z->A

Now you should have a folder structure like this:

    ├── AddTrustExternalCARoot.crt
    ├── COMODORSAAddTrustCA.crt
    ├── COMODORSADomainValidationSecureServerCA.crt
    ├── cdn_guillaumemaka_com.crt
    ├── private.key
    └── pem
        ├── AAACertificateServices.pem
        ├── CAChain.pem
        ├── USERTrustRSAAAACA.pem
        ├── SectigoRSADomainValidationSecureServerCA.pem
        ├── STAR.example.com.pem
        └── private.key.pem

  AWS ACM import 
    Certificate body only need this file : STAR.example.com.pem
    Certificate private key need this file : private.key.pem
    Certificate chain need this file : CAChain.pem


  

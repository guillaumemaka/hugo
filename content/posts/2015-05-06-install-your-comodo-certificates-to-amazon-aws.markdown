---
type: post
title: "Install your Comodo Certificates to Amazon AWS"
date: 2015-05-06 20:45:53 -0400
comments: true
share: true
tags: [Amazon, SSL, AWS, Comodo]
image:
  feature: /images/abstract-2.jpg
---

Comodo, the leading Internet Security Provider offers Free Antivirus, SSL Certificate and other Internet Security related products with complete protection. In this post I will walk you through the setup of SSL in Amazon CloudFront (the process is common to all Amazon services)

<!--more-->

AWS need that all your certificates are in PEM format. They are two main of encoding certificate:

- ``DER``: is a binary encoding of a certificate. Typically these use the file extension of ``.crt`` or ``.cert``.
- ``PEM``: is a Base64 encoding of a certificate represented in ASCII therefore it is readable as a block of text. This is very useful as you can open it in a text editor work with the data more easily.

Comodo certificate are delivered in DER format ``.crt``, so we need to convert them to ``PEM``.

# Certificates Setup

## Convert crt to PEM

Amazon AWS need:

- Your issued certificate
- Your private key
- The CAChain certificate that include all intermediate and Root CA certificate.

Comodo send you 4 certificates:

- AddTrustExternalCARoot.crt
- &lt;your_issued_certificate_name&gt;.crt: for instance ``cdn_guillaumemaka_com.crt`` in my case.
- COMODORSAAddTrustCA.crt
- COMODORSADomainValidationSecureServerCA.crt

First cding to the folder containning all your certificates:

```shell Terminal
$ cd /path/to/certificates/folder
$ mkdir pem
```

Then convert all certificates:

```shell Terminal
openssl x509 -in ./AddTrustExternalCARoot.crt -outform pem -out ./pem/AddTrustExternalCARoot.pem
openssl x509 -in ./COMODORSAAddTrustCA.crt -outform pem -out ./pem/COMODORSAAddTrustCA.pem
openssl x509 -in ./COMODORSADomainValidationSecureServerCA.crt -outform pem -out ./pem/COMODORSADomainValidationSecureServerCA.pem
openssl x509 -in ./cdn_guillaumemaka_com.crt -outform pem -out ./pem/cdn_guillaumemaka_com.pem
```

* ``x509``: The x509 command is a multi purpose certificate utility. It can be used to display certificate information, convert certificates to various forms, sign certificate requests like a "mini CA" or edit certificate trust settings.
* ``-in <filename>``: This specifies the input filename to read a certificate from or standard input if this option is not specified.
* ``-outform PEM``: This specifies the output format. In this case ``PEM``.
* ``-out filename`` : This specifies the output filename to write to or standard output by default.

Convert the private key:

```shell Terminal
openssl rsa -in ./private.key -outform PEM -out private.key.pem
```

* ``rsa``: The rsa command processes RSA keys.



## Create a CAChain

```shell Terminal
$ cat ./pem/COMODORSADomainValidationSecureServerCA.pem > ./pem/CAChain.pem
$ cat ./pem/COMODORSAAddTrustCA.pem >> ./pem/CAChain.pem
$ cat ./pem/AddTrustExternalCARoot.pem >> ./pem/CAChain.pem
```
---
{{< alert class="alert-warning" >}}
Warning: You must construct the CAChain in descending order. Z->A
{{< /alert >}}

Now you should have a folder structure like this:

```shell
├── AddTrustExternalCARoot.crt
├── COMODORSAAddTrustCA.crt
├── COMODORSADomainValidationSecureServerCA.crt
├── cdn_guillaumemaka_com.crt
├── private.key
└── pem
    ├── AddTrustExternalCARoot.pem
    ├── CAChain.pem
    ├── COMODORSAAddTrustCA.pem
    ├── COMODORSADomainValidationSecureServerCA.pem
    ├── cdn_guillaumemaka_com.pem
    └── private.key.pem
```

## Upload

```
aws iam upload-server-certificate --server-certificate-name CDNServerCertificate --certificate-body file://cdn_guillaumemaka_com.pem --private-key file://private.key.pem --certificate-chain file://CAChain.pem --path /cloudfront/production/
```

---
{{< alert class="alert-info" >}}
Notice: ``--path /cloudfront/production`` options specify that we the certificate to be available only in the CloudFront service.
{{< /alert >}}

# Bonus: Setup CloudFront HTTPS End Point

 1) Login to your [Amazon AWS Account]

{% link_img center /images/aws-image-01.png %}

 2) Go to the CloudFront shell.  

{% link_img center /images/aws-cloudfront-image-01.png %}

 3) Click on the id of your cloudfront instance.

{% link_img center /images/aws-cloudfront-image-02.png %}

 4) Click **Edit**.

{% link_img center /images/aws-cloudfront-image-03.png %}

 5) Select the option **_Custom SSL Certificate_** and select the certificate previously uploaded. Go to the bottom of the page and click **_Save_**.


{% link_img center /images/aws-cloudfront-image-04.png %}

 6) On the main page got to the **_Behaviors_** tab then click **_Create Behavior_**.

{% link_img center /images/aws-cloudfront-image-05.png %}

 7) Configure the behavior:

 {% link_img center /images/aws-cloudfront-image-06.png %}

* **_Path pattern_**: the sub path of the url you want to add a behavior.
* **_Viewer Policy_**: select Redirect HTTP to HTTPS.
* **_Allow HTTP Method_**: select GET, HEAD (I configuring a CDN, so I just need GET and HEAD request).

 8) Click **_Create_**.

That's it ! Open the url in your browser and check if the HTTP url redirect to HTTPS.

```shell
$ curl -I http://cdn.example.com/images/animage.png
HTTP/1.1 301 Moved Permanently
Server: CloudFront
Date: Sun, 10 May 2015 13:42:36 GMT
Content-Type: text/html
Content-Length: 183
Connection: keep-alive
Location: https://cdn.example.com/images/animage.png
X-Cache: Redirect from cloudfront
Via: 1.1 <id>.cloudfront.net (CloudFront)
X-Amz-Cf-Id: EGQVJRkntlSPRf4MxSBWMlt86EW6s29JekUah6fj6kMmMJFj8ugMIw==
```

# Resources

- [Stackoverflow]
- [Andrey Lagayev]
- [networkassassin.com]
- [AWS Command Line CLI Documentation]
- [IAM Documentation]
- [Heroku SSL Documentation]

[Amazon AWS Account]: https://aws.amazon.com
[Stackoverflow]:http://serverfault.com/questions/643401/cant-get-network-solutions-certificate-chain-working-with-ec2-elastic-load-bala
[Andrey Lagayev]:http://andrey.legayev.com/2013/06/openssl-convert-private-key-to-pem.html
[networkassassin.com]: http://www.networkassassin.com/openssl-commonly-used-commands/
[AWS Command Line CLI Documentation]: http://docs.aws.amazon.com/cli/latest/reference/iam/upload-server-certificate.html
[IAM Documentation]:http://docs.aws.amazon.com/IAM/latest/UserGuide/InstallCert.html
[Heroku SSL Documentation]:https://devcenter.heroku.com/articles/ssl-endpoint#acquire-ssl-certificate

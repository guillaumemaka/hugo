---
type: post
title: "Deploy Octopress to Amazon AWS with Elastic Beanstalk"
date: 2015-05-05 14:22:00 -0400
modified: 2015-06-04 06:26:00 -0400
published: true
comments: true
share: true
tags: [Web, Blogging, Octopress, Amazon, Elastic Beanstalk]
image:
  feature: /images/abstract-3.jpg
---

Amazon Web Services (AWS) comprises dozens of services, each of which exposes an area of functionality. While the variety of services offers flexibility for how you want to manage your AWS infrastructure, it can be challenging to figure out which services to use and how to provision them.

With Elastic Beanstalk, you can quickly deploy and manage applications in the AWS cloud without worrying about the infrastructure that runs those applications. AWS Elastic Beanstalk reduces management complexity without restricting choice or control. You simply upload your application, and Elastic Beanstalk automatically handles the details of capacity provisioning, load balancing, scaling, and application health monitoring. Elastic Beanstalk uses highly reliable and scalable services that are available in the [AWS Free Usage Tier].

This post will walk you through the process of deploying an Octopress blog to Amazon AWS with Amazon Elastic Beanstalk.

<!--more-->

# Install EB CLI

## On Mac OS X

### Homebrew

Open terminal and use the following command to download and run the installation script provided by Homebrew if its not already installed.

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```  

Install EB CLI

```
$ brew update && brew upgrade --all
$ brew install awsebcli
```

### Python

#### On OS X 10.7 or later

1) Open a terminal and use the following command to download and run the installation script provided by AWS.

```
$ curl -s https://s3.amazonaws.com/elasticbeanstalk-cli-resources/install-ebcli | bash
```

2) Verify that the EB CLI is installed correctly:

```
$ eb --version
EB CLI 3.2.2 (Python 3.4.3)
```

#### On OS X 10.6 or earlier

1) Install Python 3.4 from the [downloads page] of [Python.org].  

2) Install pip by using the script provided by the Python Packaging Authority.  

```
$ curl -O https://bootstrap.pypa.io/get-pip.py
$ python3 get-pip.py
```

3) Use pip to install the EB CLI.  

```
$ sudo pip install awsebcli
```

4) Verify that the EB CLI is installed correctly:  

```
$ eb --version
EB CLI 3.2.2 (Python 3.4.3)
```

-----------

{{< alert class="alert-info" >}}
The command line tools is regularly update so run this command often ``pip install awsebcli --upgrade``
{{< /alert >}}

## Linux

1) Check to see if Python is already installed:  
2) If Python 2.7 or later is not installed, install it with your distribution's package manager. The command and package name varies:  


 * On Debian derivatives such as Ubuntu, use APT:  

```
$ sudo apt-get install python2.7
```

 * On Red Hat and derivatives, use yum:  

```
$ sudo yum install python27
```

 * On SUSE and derivatives, use zypper:  

```
$ sudo zypper install python
```

3) Open a command prompt or shell and run the following command to verify that Python installed correctly:  

```
$ python --version
Python 2.7.9
```

## Windows

1) Install Python 3.4 from the [downloads page] of [Python.org].  
2) Add the Python home and scripts directories to the Windows Path system variable:  

```
C:\WINDOWS\system32;C:\WINDOWS;C:\Python34;C:\Python34\Scripts
```

3) Open the Windows Command Processor from the Start menu.  
4) Verify that Python and pip are both installed correctly with the following commands:  

```
C:\Windows\System32> python --version
Python 3.4.3
C:\Windows\System32> pip --version
pip 6.0.8 from C:\Python34\lib\site-packages (python 3.4)
```

5) Install the EB CLI using pip:  

```
C:\Windows\System32> pip install awsebcli
Collecting awsebcli
  Downloading awsebcli-3.2.2.tar.gz (828kB)
Collecting pyyaml>=3.11 (from awsebcli)
  Downloading PyYAML-3.11.tar.gz (248kB)
Collecting cement==2.4 (from awsebcli)
  Downloading cement-2.4.0.tar.gz (129kB)
Collecting python-dateutil<3.0.0,>=2.1 (from awsebcli)
  Downloading python_dateutil-2.4.2-py2.py3-none-any.whl (188kB)
Collecting jmespath>=0.6.1 (from awsebcli)
  Downloading jmespath-0.6.2.tar.gz
Collecting six>=1.5 (from python-dateutil<3.0.0,>=2.1->awsebcli)
  Downloading six-1.9.0-py2.py3-none-any.whl
Installing collected packages: six, jmespath, python-dateutil, cement, pyyaml, awsebcli
  Running setup.py install for jmespath
  Running setup.py install for cement
  Running setup.py install for pyyaml
    checking if libyaml is compilable
    Microsoft Visual C++ 10.0 is required (Unable to find vcvarsall.bat).
    skipping build_ext
  Running setup.py install for awsebcli
    Installing eb-script.py script to C:\Python34\Scripts
    Installing eb.exe script to C:\Python34\Scripts
    Installing eb.exe.manifest script to C:\Python34\Scripts
Successfully installed awsebcli-3.2.2 cement-2.4.0 jmespath-0.6.2 python-dateutil-2.4.2 pyyaml-3.11 six-1.9.0
```

6) Verify that the EB CLI is installed correctly:  

```
C:\Windows\System32> eb --version
EB CLI 3.2.2 (Python 3.4.3)
```

# Initialize and deploy your blog

## Initialize an EB environment

1) Open a terminal and enter the following command

```
$ cd /path/to/your/blog
$ eb init
```

2) You will be prompt to answer some question like those one below:

```
Select a default region

1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-southeast-1 : Asia Pacific (Singapore)
7) ap-southeast-2 : Asia Pacific (Sydney)
8) ap-northeast-1 : Asia Pacific (Tokyo)
9) sa-east-1 : South America (Sao Paulo)
(default is 3): 1

Enter Application Name
(default is "octopress"):
Application octopress has been created.

It appears you are using Ruby. Is this correct?
(y/n): y

Select a platform version.
1) Ruby 2.2 (Puma)
2) Ruby 2.1 (Puma)
3) Ruby 2.0 (Puma)
4) Ruby 2.2 (Passenger Standalone)
5) Ruby 2.1 (Passenger Standalone)
6) Ruby 2.0 (Passenger Standalone)
7) Ruby 1.9.3
(default is 1): 1

# You can skip the set up for SSH

Do you want to set up SSH for your instances?
(y/n): y

# Not asked if you enter \"n\" for the previous question

Select a keypair.
1) EC2InstanceBlogSSHKeyPair
2) [ Create new KeyPair ]
(default is 2): 1

```

## Create and deploy

### Create a load balanced environment

Run this command and follow the instructions, for example:

```
$ eb create
Enter Environment Name
(default is octopress): octopress-personal-blog
Enter DNS CNAME prefix
(default is octopress-personal-blog): guillaumemaka-octopress-personal-blog
Creating application version archive "v2_0-279-g388b".
Uploading: [##################################################] 100% Done...
Environment details for: octopress-personal-blog
  Application name: octopress-personal-blog
  Region: us-east-1
  Deployed Version: v2_0-279-g388b
  Environment ID: e-bb3eiiv3nq
  Platform: 64bit Amazon Linux 2015.03 v1.3.1 running Ruby 2.2 (Puma)
  Tier: WebServer-Standard
  CNAME: guillaumemaka-octopress-personal-blog.elasticbeanstalk.com
  Updated: 2015-05-05 12:21:18.165000+00:00
Printing Status:
INFO: createEnvironment is starting.
INFO: Using elasticbeanstalk-us-east-1-615777031639 as Amazon S3 storage bucket for environment data.
INFO: Created security group named: sg-eb1e478f
INFO: Created load balancer named: awseb-e-b-AWSEBLoa-1P21WJ5KMYTSO
INFO: Created security group named: awseb-e-bb3eiiv3nq-stack-AWSEBSecurityGroup-1TISVFD070EPR
INFO: Created Auto Scaling launch configuration named: awseb-e-bb3eiiv3nq-stack-AWSEBAutoScalingLaunchConfiguration-1UUJG9ZZQZZZL
INFO: Created Auto Scaling group named: awseb-e-bb3eiiv3nq-stack-AWSEBAutoScalingGroup-33CM6T10BZ3L
INFO: Waiting for EC2 instances to launch. This may take a few minutes.
INFO: Created Auto Scaling group policy named: arn:aws:autoscaling:us-east-1:615777031639:scalingPolicy:24164845-7e5d-4098-a55b-37495656d623:autoScalingGroupName/awseb-e-bb3eiiv3nq-stack-AWSEBAutoScalingGroup-33CM6T10BZ3L:policyName/awseb-e-bb3eiiv3nq-stack-AWSEBAutoScalingScaleDownPolicy-1DXOP5XH0C992
INFO: Created Auto Scaling group policy named: arn:aws:autoscaling:us-east-1:615777031639:scalingPolicy:33c0aa56-24f8-495d-a83d-2ad9fe2b1004:autoScalingGroupName/awseb-e-bb3eiiv3nq-stack-AWSEBAutoScalingGroup-33CM6T10BZ3L:policyName/awseb-e-bb3eiiv3nq-stack-AWSEBAutoScalingScaleUpPolicy-JJTP8T9N0XJH
INFO: Created CloudWatch alarm named: awseb-e-bb3eiiv3nq-stack-AWSEBCloudwatchAlarmLow-1I2JXAT8DN7SO
INFO: Created CloudWatch alarm named: awseb-e-bb3eiiv3nq-stack-AWSEBCloudwatchAlarmHigh-18720H7ELBH0W
INFO: Added EC2 instance 'i-2a51b6d5' to Auto Scaling Group 'awseb-e-bb3eiiv3nq-stack-AWSEBAutoScalingGroup-33CM6T10BZ3L'.
INFO: Application available at guillaumemaka-octopress-personal-blog.elasticbeanstalk.com.
INFO: Successfully launched environment: octopress-personal-blog
```

### Create a Single instance environment

Run this command

```
$ eb create octopress-personal-blog  --region us-east-1 -instance_type t1.micro --single
```

Where:

* **_region_**: the region where you want to deploy.
* **_instance_type_**: the instance type you want to deploy.
* **_single_**: ask only to use a single instance instead of a load balanced environment.

#### **Optional** Configure SSL for a Single Instance

Create an hidden folder called **.ebextensions** at the top root level directory of your blog

```
cd /path/to/your/blog
$ mkdir .ebextensions

```

Create a **.config** file, for instance **singleinstancessl.config** and put these line into it:

```
Resources:
  sslSecurityGroupIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupName: <REPLACE_WITH_YOUR_EC2_GROUPNAME>
      IpProtocol: tcp
      ToPort: 443
      FromPort: 443
      CidrIp: 0.0.0.0/0

files:
  /etc/nginx/conf.d/ssl.conf:
    content: |
      # HTTPS server

      server {
          listen       443;
          server_name  localhost;

          ssl                  on;
          ssl_certificate      /etc/pki/tls/certs/server.crt;
          ssl_certificate_key  /etc/pki/tls/certs/server.key;

          ssl_session_timeout  5m;

          ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
          ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
          ssl_prefer_server_ciphers   on;

          location / {
              proxy_pass  http://my_app;
              proxy_set_header        Host            $host;
              proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
          }

          location /assets {
            alias /var/app/current/public/assets;
            gzip_static on;
            gzip on;
            expires max;
            add_header Cache-Control public;
          }

          location /public {
            alias /var/app/current/public;
            gzip_static on;
            gzip on;
            expires max;
            add_header Cache-Control public;
          }
      }

  /etc/pki/tls/certs/server.crt:
    content: |
      -----BEGIN CERTIFICATE-----
      your-certificate-here
      -----END CERTIFICATE-----

  /etc/pki/tls/certs/server.key:
    content: |
      -----BEGIN RSA PRIVATE KEY-----
      your-key-here
      -----END RSA PRIVATE KEY-----

container_commands:
  01restart_nginx:
    command: "service nginx restart"

```

1. You need to replace the **GroupName** by the current EC2 security group name created by previous command. (You can find the group name in the EC2 dashboard > Security Groups)

2. For certificates paste the content of your certificate between **-----BEGIN CERTIFICATE-----** and **-----END CERTIFICATE-----** and the content of your private key between **-----BEGIN RSA PRIVATE KEY-----** and **-----END RSA PRIVATE KEY-----**.

## Deplay

Run this command to deploy your blog

```
$ eb deploy
```

## Verify that your blog is up and running


```
$ eb open
```

# Resources

- [AWS Free Usage Tier]
- [EB CLI Installation Documentation]
- [EB CLI Reference Documentation]
- [Elastic Beanstalk Documentation]
- [Homebrew]
- [Python.org]

[AWS Free Usage Tier]: http://aws.amazon.com/free
[downloads page]: https://www.python.org/downloads/
[Python.org]: https://www.python.org
[EB CLI Installation Documentation]: http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html#eb-cli3-install-linux
[EB CLI Reference Documentation]: http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-reference-eb.html
[Homebrew]: http://brew.sh/
[Elastic Beanstalk Documentation]: http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html

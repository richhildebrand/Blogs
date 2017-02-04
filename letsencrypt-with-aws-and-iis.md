## Introduction

[Let's Encrypt](https://letsencrypt.org) provides free SSL/TSL certificates. AWS also provides free certificates, but only if you are willing to pay $20 a month to use their loadbalencer. In my case I wanted to avoid this costs and use a single EC2 instance created through elastic beanstalk.


## Remote into your EC2 server

To enable remote desktop into your elastic beanstalk server, you will need to first create an EC2 key pair and assign it your server. You will also need to configure the EC2's security group to allow RDP from your IP address via TCP on port 3389. The documentation AWS provides for this, while not perfect, should be sufficient to guide you through this process.


## Installing ACMESharp

ACMESharp is a powershell library created to simplify the letsecrypt verification process. Installing it was a bit tricky since the most current EC2 windows server does not support PowerShell Gallery.

First, download the latest version of [ACME-posh.zip](https://github.com/ebekker/ACMESharp/releases). Version 0.8.1.0 was used when writing this article.

Now extract the contents of the zip file to `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\ACMESharp`

Additionally, this install is currently missing the files `ACMESharp.Providers.IIS.dll` and `ACMESharp.Providers.IIS.pdb`. You will need to manaully aquire these files and place them in the directory`C:\Windows\System32\WindowsPowerShell\v1.0\Modules\ACMESharp`.

I was able to accomplish this by installing ACMESharp via the PowerShell Gallery on a Windows 10 machine.


## Download Certify GUI

Now let us install the [certify GUI](https://certify.webprofusion.com/home/download) to make things a little easier. You can use ACMESharp directly with powershell, but the errors messages are a bit vague.


## Using Certify GUI

This can be done with four pretty quick steps

1) Using IIS Manager, add your domain name to your IIS bindings for port 80.
2) Using Certify, file -> new -> Contact Registration
3) Using Certify, file -> new -> Domain Certificate
4) Using IIS Manager, add a default IIS binding for port 443.

## Allow HTTPS traffic to your E2C serer

Finally, create a new file, `htts.config` in your `.ebextensions` folder. It should contain the following code.

    Resources:
      sslSecurityGroupIngress:
        Type: AWS::EC2::SecurityGroupIngress
        Properties:
          GroupId: {"Fn::GetAtt" : ["AWSEBSecurityGroup", "GroupId"]}
          IpProtocol: tcp
          ToPort: 443
          FromPort: 443
          CidrIp: 0.0.0.0/0

More information on what this does can be found [here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/https-singleinstance.html).

## Important information

While your letsencrypt certificate is free, it will need to be renewed in 90 days, but can be renewed sooner than that. My advice would be to renew it at day 60.

These instructions also do not force HTTPS, I will cover that in my next post.
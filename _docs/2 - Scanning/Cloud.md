---
title: Enumerating Cloud   
category: Scanning
order: 99
---

> **AWS Buckets**

[slurp](https://github.com/bbb31/slurp) | <code> slurp domain -t domain.com or slurp keyword -t domain </code>
[Bucket Finder](https://digi.ninja/files/buck_finder_1.1.tar.bz2) | <code> bucket_finder.rb --region us wordlist.txt --download </code>
AWS CLI Permissions | <code> aws s3api get-bucket-acl --bucket nameofbucket </code>
AWS CLI Read files | <code> aws s3 ls s3://bucketname </code>
AWS CLI Write File | <code> aws s3 mv file.txt s3://bucketname </code>
AWS CLI File Permissions | <code> aws s3api get-object-acl --bucket bucketname --key file.txt </code>

**Resources:**

* [AWS access control](https://labs.detectify.com/2017/07/13/a-deep-dive-into-aws-s3-access-controls-taking-full-control-over-your-assets)

**Kerberos:**

NMAP | <code> nmap -p88 --script krb5-enum-users --script-args krb5-enum-users.realm="domain.com",userdb=/opt/userlist.txt [domain controler IP] </code>

> **Subdomain Takeovers**

tko-subs | <code> tkosubs -domains=list.txt -data=providers-data.csv -output=output.csv </code>
[HostileSubBruteforcer](https://github.com/nahamsec/HostileSubBruteforcer) | <code>TBD</code>
[autoSubTakeover](https://github.com/JordyZomer/autoSubTakeover) |  <code> TBD </code>

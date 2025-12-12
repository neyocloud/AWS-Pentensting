# AWS-Pentensting
Pentesting AWS Misconfigurations: A Hands-On Walkthrough with flaws.cloud


# My Journey Through the flaws.cloud AWS Challenge (Levels 1–3)

Recently, I completed the first three levels of the flaws.cloud AWS security challenge. This challenge involves uncovering misconfigurations in AWS S3 buckets and related services. In this walkthrough, I will describe how I set up my environment (including AWS CLI and IAM credentials) and stepped through each level, discovering hidden secrets and understanding the security lessons along the way. All steps are written from my perspective, as I performed them, complete with the actual commands I ran and their outputs.

Environment Setup and AWS CLI Configuration

Before diving into the challenge, I prepared my AWS environment and tools:

AWS Account and IAM User: I made sure I had access to an AWS account (free tier is sufficient). To avoid using root credentials, I created a new IAM user with programmatic access (Access Key ID and Secret Access Key). This user was granted necessary permissions for S3 operations. Having a dedicated IAM user for the challenge is important for security and helps simulate “any authenticated AWS user” scenarios

AWS CLI Installation: I installed the latest AWS Command Line Interface (AWS CLI v2) on my local machine and verified it was installed by checking the version running aws --version in the terminal

AWS CLI Configuration: Using the credentials from the IAM user, I configured the AWS CLI on my machine. I ran aws configure (or aws configure --profile [profile-name] for a named profile) and entered the Access Key ID, Secret Access Key, preferred region, and output format when prompted. This saved my credentials locally so I could use AWS CLI commands seamlessly. With this setup complete, I was ready to interact with AWS services from my terminal.

## Level 1: Enumerating a Public S3 Bucket



Level 1 of the challenge presented the domain flaws.cloud. The goal was to figure out what this domain was and find a hidden "secret" file. The hint suggested this level would be "buckets of fun," which immediately made me suspect an Amazon S3 bucket might be involved.

DNS Enumeration: I began by investigating the domain. Using a DNS lookup tool (nslookup on my Mac terminal), I checked the IP address associated with flaws.cloud. Then I performed a reverse DNS lookup on that IP to see if it pointed to an AWS service. Indeed, the reverse lookup revealed an address in the format s3-website-us-west-2.amazonaws.com, confirming that flaws.cloud was hosted as an S3 static website in the us-west-2 region


<img width="531" height="430" alt="image" src="https://github.com/user-attachments/assets/e706fa45-6fe7-4333-b97c-9359fb1c189b" />




This told me that an S3 bucket (likely named "flaws.cloud") was serving the website content.






## 2. Listing the S3 Bucket Contents: Knowing the bucket’s region and name, I attempted to list its contents using AWS CLI. I ran:

```
 aws s3 ls s3://flaws.cloud --no-sign-request

```

I got a directory listing of the bucket’s contents. The bucket was world-readable, as expected for a publicly accessible S3 website. I saw several files listed, including some HTML hint files and a file that caught my attention: secret.html (or similarly named). This was likely the secret file I needed to find.

<img width="497" height="116" alt="image" src="https://github.com/user-attachments/assets/73d3436d-5df9-4d98-bda0-cacef918891a" />



## Retrieving the Secret File: I used the AWS CLI to download the secret file for closer inspection:


```

 aws s3 cp s3://flaws.cloud/secret-dd02c7c.html --no-sign-request thesecret.html 
download: s3://flaws.cloud/secret-dd02c7c.html to ./thesecret.html
```


This copied the file to my local machine. I then opened thesecret.html in my web browser to see its content.

Content of the secret file from Level 1, revealing the URL for Level 2. The file contained a congratulatory message and the address of the next challenge level (notice the level2-<hash>.flaws.cloud URL).


<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/0de34f42-ab45-41a6-beaf-d292f4296324" />


As shown in the screenshot above, the secret file contained a congratulations message for finding it, along with a URL for Level 2 of the challenge. I moved on to the next level.



## Level 2: Authenticated Access to an S3 Bucket

Level 2 introduced a twist: the new subdomain (Level 2 URL) was not openly browseable like the first. I suspected this bucket might be configured to only allow access to “authenticated users” – an AWS misconfiguration where any AWS account user can access the bucket, but the general public cannot


The challenge prompt even hinted that an AWS account would be required, which is why I set up my AWS IAM user and CLI earlier.

DNS Analysis of Level 2 Domain: Similar to Level 1, I started by examining the new domain (level2.flaws.cloud). Using nslookup again, I found that this domain also resolved to a set of IP addresses in AWS. I then did a reverse lookup on one of those IPs to determine the service behind it.

DNS lookup for the Level 2 subdomain. The nslookup result shows multiple IP addresses, indicating the domain is hosted on AWS infrastructure【5†】.

Reverse DNS lookup of one IP from the Level 2 domain. The PTR record reveals the host is an S3 website endpoint in the us-west-2 region【6†】, confirming that the Level 2 subdomain corresponds to an Amazon S3 bucket (just like Level 1).


<img width="751" height="530" alt="image" src="https://github.com/user-attachments/assets/412ed82d-1b98-4f24-8f19-9a113bb853d7" />



The DNS checks confirmed that Level 2 is also an S3 bucket, likely configured as a website in the same region. The difference was that browsing to the URL or listing it anonymously might not work if the bucket’s policy was restrictive (e.g., accessible only to authenticated AWS users).

Attempting to List Bucket Contents (with AWS CLI): Using the AWS CLI with the credentials I configured, I attempted to list the contents of the Level 2 bucket. I ran:

```
aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud

```


Attempting to list the Level 2 bucket via AWS CLI. The first attempt (with my credentials) returned "AccessDenied". The second attempt with --no-sign-request succeeded, listing the bucket’s files【7†】. The output shows files.........




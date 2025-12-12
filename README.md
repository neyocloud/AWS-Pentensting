# AWS-Pentensting
Pentesting AWS Misconfigurations: A Hands-On Walkthrough with flaws.cloud


# My Journey Through the flaws.cloud AWS Challenge (Levels 1–3)

Recently, I completed the first three levels… along the way. All steps are written from my perspective, as I performed them…

Environment Setup and AWS CLI Configuration

Before diving into the challenge, I prepared my AWS environment and tools:

AWS Account and IAM User: I made sure I had access to an AWS account (free tier is sufficient). To avoid using root credentials, I created a new IAM user with programmatic access (Access Key ID and Secret Access Key). This user was granted necessary permissions for S3 operations. Having a dedicated IAM user for the challenge is important for security and helps simulate “any authenticated AWS user” scenarios

AWS CLI Installation: I installed the latest AWS Command Line Interface (AWS CLI v2) on my local machine and verified it was installed by checking the version running aws --version in the terminal


<img width="446" height="51" alt="image" src="https://github.com/user-attachments/assets/6b31c8ba-72e9-4f7e-8c25-e41c1cd8f9e5" />



AWS CLI Configuration: Using the credentials from the IAM user, I configured the AWS CLI on my machine. I ran aws configure (or aws configure --profile [profile-name] for a named profile) and entered the Access Key ID, Secret Access Key, preferred region, and output format when prompted. This saved my credentials locally so I could use AWS CLI commands seamlessly. With this setup complete, I was ready to interact with AWS services from my terminal.

## Level 1: Enumerating a Public S3 Bucket



Level 1 of the challenge presented the domain flaws.cloud. The goal was to figure out what this domain was and find a hidden "secret" file. The hint suggested this level would be "buckets of fun," which immediately made me suspect an Amazon S3 bucket might be involved.

DNS Enumeration: I began by investigating the domain. Using a DNS lookup, I checked the IP address associated with flaws.cloud. Then I performed a reverse DNS lookup on that IP to see if it pointed to an AWS service. Indeed, the method revealed an address in the format s3-website-us-west-2.amazonaws.com, confirming that flaws.cloud was hosted as an S3 static website in the us-west-2 region


**The DNS mapping exposes the underlying AWS architecture, region, and storage service — giving you everything you need to start probing the bucket.*


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

<img width="776" height="115" alt="image" src="https://github.com/user-attachments/assets/37b310ed-77df-4f8a-abd0-fb5b9651a7fa" />


following the next step.....


```

 aws s3 cp \
s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/secret-e4443fc.html \
./thesecret2.html \
--no-sign-request

```
Then i opened using 

```

 open thesecret2.html

```

<img width="1253" height="735" alt="image" src="https://github.com/user-attachments/assets/bbd3f8d6-bf74-4105-b690-2426f300516f" />



Repeating the usual steps

<img width="757" height="453" alt="image" src="https://github.com/user-attachments/assets/8f276e9f-1ad5-40d8-9138-4c3c91d57dfa" />



```

 aws s3 ls s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud

```



Downloading the Entire Bucket (Repository): Given the presence of a Git repository, I decided to download the entire bucket contents for analysis. Instead of fetching files one-by-one, I used the convenient sync command:

mkdir thecretstuff && cd thecretstuff

```
 aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud ./thecretstuff
```


<img width="1380" height="523" alt="image" src="https://github.com/user-attachments/assets/d7afa1e0-359e-43c5-89ce-b818db8c67d1" />


This command recursively downloaded all files from the bucket into a local directory (thecretstuff). The output (shown in the screenshot above) logged each file as it was downloaded, including many files under a .git directory (e.g., objects, refs, config, etc.). Essentially, I now had a local copy of the website’s Git repository.



<img width="1113" height="252" alt="image" src="https://github.com/user-attachments/assets/16fd562f-c1e3-4aa5-a5b7-7f00a7cfb2ab" />


Went further to use the command tree

<img width="541" height="913" alt="image" src="https://github.com/user-attachments/assets/9d247c79-f1a6-4a5b-b736-03929a98ba89" />



Investigating the Git Repository: With the repository on hand, I opened a terminal in the thecretstuff directory (which now was a Git repo) and ran standard git commands to see the history:


<img width="1404" height="597" alt="image" src="https://github.com/user-attachments/assets/394247af-dad3-43d9-ab22-fd7e77c85caf" />



And yes i found both the access key and the secret key

git log — This showed the commit history of the project. I looked at the commit messages and IDs to identify any that might contain secrets.

git checkout <commit_hash> — I checked out earlier commits by their hashes to see the state of the repository at those points in time.

By examining the diffs and files in those commits, I eventually found a file (e.g., access_keys.txt in one commit) that contained AWS Access Key ID and Secret Access Key for a user

So i configured my aws profile gotcha using their access key and secret key .......


made use of these commands 

```
 aws s3 ls --profile gotcha s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud
```

```
 aws s3 ls --profile gotcha

```
<img width="955" height="290" alt="image" src="https://github.com/user-attachments/assets/1201181d-1ee5-4052-9492-cff234561210" />


To anyone else, it might look like a bunch of random files.
But to me, it felt like stepping into a developer’s forgotten workshop clues everywhere, waiting for someone curious enough to follow them.

The more buckets I listed, the clearer it became:
This wasn’t just storage.
It was a roadmap of misconfigurations — a perfect training ground for learning how AWS mistakes lead to real vulnerabilities.


**These were the leaked credentials mentioned in the challenge goal. The challenge had cleverly hidden AWS keys in a previous state of the code repository, assuming that leaving a .git directory publicly accessible would expose that history.*


## Notes


DNS enumeration can reveal AWS hosting patterns

Public buckets can leak sensitive files

.git directories should never be served over HTTP

Old commits can expose historical secrets



Challenge Outcome and Lessons: Obtaining the AWS keys from the repository essentially completed Level 3. The presence of those credentials demonstrated why one must never expose sensitive files or leave version control repositories in public web roots. I learned that the leaked keys belonged to a vulnerable IAM user set up for the challenge. In a real scenario, such keys could be used by an attacker to access resources. As a precaution, whenever credentials are exposed, they should be immediately revoked to prevent exploitation

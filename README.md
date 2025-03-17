<img width="674" alt="11" src="https://github.com/user-attachments/assets/202ab6a4-8644-4bdb-872e-ab4a0ab4cad2" /># aws-challenge-week-4
Week 4 of the 12 weeks of AWS challenge

The fourth week of the challenge focused on storage. AWS offers a complete range of services for you to store, access, govern and analyze your data. In AWS you can select from a wide range of options, such as object storage, file storage and block storage services, backup and data migration options to build the foundation of your cloud IT environment. 

## Prerequisites

Before every hands on lab, use AWS CloudFormation to deploy the necessary resources as code, the templates will be used for setting up the workshop ennvironment in us-west-2 region. The templates are the following:
* sid-base-template.yaml (required for all of the labs)
* sid-backup-lab.yaml
* sid-datamigration-onprem.yaml
* sid-performance.yaml
* sid-s3-security.yaml

## S3 Security best practices

In the hands on labs, I will be doing the following:
* Use bucket policies, encryption and VPC endpoints to secure my S3 buckets
* Use AWs Config and IAM Access analyzer to review my S3 security posture

### Prerequisite steps
1. Attach an IAM role to the running EC2 instance

<img width="958" alt="2" src="https://github.com/user-attachments/assets/ab34d4b6-2777-4f07-aa2b-9ab66ffb3203" />

2. Connect to the EC2 instance using EC2 instance connect and verify that you can access the S3 buckets in the account

<img width="289" alt="3" src="https://github.com/user-attachments/assets/12db9ede-fe32-438f-9409-1a165c24e304" />

3. Create a S3 bucket policy that requires connections to use HTTPS

<img width="956" alt="4" src="https://github.com/user-attachments/assets/497f58fb-e775-4ef8-ac1d-b473b3d95743" />

4. Run the following command in SSH session The command returns a 403 error because the endpoint-url is HTTP

<img width="875" alt="5" src="https://github.com/user-attachments/assets/faeba816-b501-4010-b287-cab92c3bd3ab" />

5. Run another command in the SSH session. This command succeeds because the s3api used HTTPS.

<img width="869" alt="6" src="https://github.com/user-attachments/assets/63e55848-7e70-4903-a988-fa277ea75a21" />

6. Create a KMS key and assign it the permissions for encryption and decryption

<img width="956" alt="7" src="https://github.com/user-attachments/assets/c591eaa3-9a72-417c-a20d-fa2b9ab1d7c1" />

7. Open the SSH session and run the following command to create a test file

<img width="443" alt="8" src="https://github.com/user-attachments/assets/c1be9557-198f-4dab-98d9-8892494b13aa" />

8. Run another command to put text01 object inside of the S3 bucket. The request is successful because the buckets current default encryption is AES256. 

<img width="707" alt="9" src="https://github.com/user-attachments/assets/d8317fa4-44f6-40b7-b622-94c9cce2461e" />

9. Update the bucket encryption to use SSE-KMS so that new objects will inherit SSE-KMS as its default encryption.

<img width="957" alt="10" src="https://github.com/user-attachments/assets/647f2a8c-51e9-4e58-9caa-c4cb9ce36931" />

10. Run another command in the SSH session to put a new text02 object into the bucket. The request is successful, server side encryption is now aws:kms instead of AES256.

<img width="674" alt="11" src="https://github.com/user-attachments/assets/4afccc7e-3c39-40d8-8b1c-754c5c994540" />

11. Run commands to compare text01 to text02

<img width="656" alt="12" src="https://github.com/user-attachments/assets/a33b600d-cb1c-4458-858c-642500b62424" />

12. 



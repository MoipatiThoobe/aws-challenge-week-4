# aws-challenge-week-4
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

11. Run commands to compare text01 to text02. text01 was written before the bucket default was changed, so it uses AES256(SSE-S3). text02 was written after we changed the bucket default, so it inherited the aws:kms (SSE-KMS) encryption.

<img width="656" alt="12" src="https://github.com/user-attachments/assets/a33b600d-cb1c-4458-858c-642500b62424" />

12. Update the bucket policy, the new policy encoforces that any new objects placed into ssekms-only/ prefix in the bucket must have SSE-KMS encryption

<img width="950" alt="13" src="https://github.com/user-attachments/assets/4a412c74-3000-4753-8ad0-11f7e9de7edf" />

13. Run comamnds in the SSH session to put text03, text04 and text05 objects into the bucket. The behaviour is as follows:
* put object for text03 will succeed and is aws:kms because it inherits the default encryption of SSE-KMS as no encryption method has been specified
* put object for text04 will succeed and it AES256 because we specified to use SSE-S3
* put object for text05 will succeed and it is aws:kms 

<img width="893" alt="14" src="https://github.com/user-attachments/assets/23c69de5-6dba-48ff-9ffd-7fef2f1e2e08" />

14. Run commands to create new objects under the prefix ssekms-only/*, the behaviour is as follows:
* put object for text06 will fail because although we have default encryption as SSE-KMS, the bucket policy requires explicit definition for SSE-KMS
* put pbject for text07 will fail because since AES256 will use SSE-S3 and we only permit SSE-KMS encryption in this prefix per the bucket policy
* put object for text08 will succeed, since SSE-KMS encryption is explicitly defined and allowed by the bucket policy

<img width="932" alt="15" src="https://github.com/user-attachments/assets/1ffeebec-c26c-49d1-ba21-a89b2b8615f5" />

15. Create an S3 VPC Endpoint

<img width="950" alt="16" src="https://github.com/user-attachments/assets/a11ea8a2-61e0-4bf4-8d23-d3b8cb01e666" />

16. Update the bucket policy with a new policy that limits access to only requests that pass through the VPC endpoint

<img width="953" alt="17" src="https://github.com/user-attachments/assets/0b7247d2-ed87-4f04-b17c-cba3bc8650a2" />

17. In the SSH session, run the following command. This command fails because the EC2 instance is accessing S3 via the internet gateway. The VPC endpoint does not yet have a route table association.

<img width="605" alt="18" src="https://github.com/user-attachments/assets/955e95ee-02b3-4acf-850d-cf859eeb1f11" />

18. Modify the route table to asscociate with the VPC endpoint so that the EC2 instance can access S3 via the VPC endpoint

<img width="952" alt="19" src="https://github.com/user-attachments/assets/715bd43e-d736-477f-a5b0-3b51e94b0dcc" />

19. In the SSH session run the command. The request will be successful because the EC2 instance is able to route its request via the VPC endpoint

<img width="598" alt="20" src="https://github.com/user-attachments/assets/7b96d893-bd07-4204-bd4b-3d460fd351ef" />

20. Configure a rule that searches for S3 buckets that are publically accessible using AWS Config. All of the rules are compliant. 

<img width="646" alt="21" src="https://github.com/user-attachments/assets/ae056924-6fb1-45ed-a7f0-e2345377a987" />

21. Update the bucket policy, the new policy will allow public read of the text01 object. An error will appear because the default block public access is enabled for the bucket.

<img width="715" alt="22" src="https://github.com/user-attachments/assets/ff5891aa-f150-4f48-9036-29ab4200a09c" />

22. Disable block public access

<img width="950" alt="23" src="https://github.com/user-attachments/assets/762562fd-537f-4279-8cff-aa45f44c66a3" />

23. We are now able to successfully update the bucket policy

<img width="959" alt="24" src="https://github.com/user-attachments/assets/a2ceb39c-8dbb-43b8-b2fe-83f6c925e01b" />

24. The AWS config rule is now showing as Non compliant

<img width="782" alt="25" src="https://github.com/user-attachments/assets/f47a4f7a-eea4-4368-801c-57f975e02bcb" />

25. Create an Access analyzer to identify buckets that are public

<img width="953" alt="26" src="https://github.com/user-attachments/assets/f6394821-3818-4b08-90ef-2339841aec02" />

26. Finding of the Access Analyzer

<img width="959" alt="27" src="https://github.com/user-attachments/assets/181b965b-af25-4528-b585-b936ac6d87c1" />

27. Delete the bucket policy that allows public access to objects within the bucket

<img width="959" alt="28" src="https://github.com/user-attachments/assets/1e608fae-14fb-4a2f-882d-3ef5947c5526" />

28. After a few minutes, the access analyzer status is changed to resolved

<img width="941" alt="29" src="https://github.com/user-attachments/assets/bacce3cd-7305-4edc-83ee-291a4aff14e0" />

29. Clean up resources

29.1 Edit the default encryption of S3 buckets to Server-side encryption with Amazon S3 managed keys (SSE-S3)

29.2 Delete the CloudFormation stacks

## Storage performance lab

In the hands on labs, I will be doing the following: 
* Test the performance of S3 by optimizing throughput, using the SYNC command and optmizing small file operations and copy operations
* Test the performance of EFS

### S3 Storage performance

1. Deploy the CloudFormation stack

<img width="959" alt="1" src="https://github.com/user-attachments/assets/fb5b6583-d845-45f7-9c2c-c2af4f7df778" />

2. Connect to the EC2 Instance using EC2 instance connect and configure the EC2 instance

<img width="320" alt="2" src="https://github.com/user-attachments/assets/474075a5-6fa3-4df3-be9e-d1734b88dec6" />

3. In the SSH session, run a command to configure the AWS CLI S3 settings

<img width="578" alt="3" src="https://github.com/user-attachments/assets/59a7f316-6eca-4c91-ad35-fd4a346295fd" />

4. Run a command to confirm the CLI configuration

<img width="334" alt="4" src="https://github.com/user-attachments/assets/e0077716-0b71-4a2c-8e8d-9cbf131cd715" />

5. Run a command to create a 1GB file

<img width="560" alt="5" src="https://github.com/user-attachments/assets/1cb790e6-20ca-487d-8af1-14b9a77358ec" />

6. Run a command to upload the file to the S3 bucket using 1 thread. The operation took 25 seconds to complete. 

<img width="623" alt="6" src="https://github.com/user-attachments/assets/1b803759-967f-4dbc-a306-94066bb06830" />

7. Run a command to upload the file to the S3 bucket using 2 threads. The operation took 14 seconds to complete.

<img width="604" alt="7" src="https://github.com/user-attachments/assets/b7c3b4a3-e196-410d-ae41-fc7101292981" />

8. Run a command to upload the file to the S3 bucket using 10 thread. The operation took 4 seconds to complete.

<img width="606" alt="8" src="https://github.com/user-attachments/assets/1fdbb29a-d054-43cc-8dda-9b9f5dda39db" />

9. Run a command to upload the file to the S3 bucket using 20 thread. The operation took 4 seconds to complete.

<img width="608" alt="9" src="https://github.com/user-attachments/assets/8ccfdcc6-42ef-4e79-8eed-b39d9217823e" />

10. Run a command to create a 200MB file

<img width="594" alt="10" src="https://github.com/user-attachments/assets/838d7182-b9e1-49ee-92df-370effe8f198" />

11. Run a command to upload 1GB of data by uploading 5 200MB files in parellel. The operation took 2 seconds to complete.

<img width="881" alt="11" src="https://github.com/user-attachments/assets/9d98c8c4-a9ab-4330-b706-9c9a06963a0b" />

12. Run a command to perform sync using 1 thread, the operation took 3 mins 23 secs to complete:
aws configure set default.s3.max_concurrent_requests 1 time aws s3 sync /ebs/tutorial/data-1m/ s3://${bucket}/sync1/

<img width="134" alt="12" src="https://github.com/user-attachments/assets/ca23c6c3-d79a-45a5-9f4c-89180471cf40" />

13. Run a command to perform sync using 10 thread, the operation took 22 seconds to complete:
aws configure set default.s3.max_concurrent_requests 10 time aws s3 sync /ebs/tutorial/data-1m/ s3://${bucket}/sync2/

<img width="119" alt="13" src="https://github.com/user-attachments/assets/8c615ea2-587d-4875-9fa6-6f73f55d9168" />

14. Run a command to create a text file that represents a list of object ids

<img width="237" alt="14" src="https://github.com/user-attachments/assets/53fb59c8-a1ee-4f35-8f9f-16a8ba5f5123" />

15. Run a command to create a 1kb file

<img width="561" alt="15" src="https://github.com/user-attachments/assets/34f4fb94-013d-4bc1-be29-cbb28987cdbc" />

16. Run a command to upload 500 1kb files to S3 using 5 threads, the operation took 49 seconds to complete:
time parallel --will-cite -a object_ids -j 5 aws s3 cp 1KB.file s3://${bucket}/run1/{}

<img width="121" alt="16" src="https://github.com/user-attachments/assets/dc076f1e-0cd2-4271-9db9-1a4f5025f1f3" />

17.  Run a command to upload 500 1kb files to S3 using 15 threads, the operation took 23.9 seconds to complete:
time parallel --will-cite -a object_ids -j 15 aws s3 cp 1KB.file s3://${bucket}/run2/{}

<img width="131" alt="17" src="https://github.com/user-attachments/assets/fe208a97-ffbd-4cdf-8ec7-0e96615341be" />

18. Run a command to upload 500 1kb files to S3 using 50 threads, the operation took 22.6 seconds to complete:
time parallel --will-cite -a object_ids -j 50 aws s3 cp 1KB.file s3://${bucket}/run3/{}

<img width="136" alt="18" src="https://github.com/user-attachments/assets/94cf6718-1c0c-4183-ad53-051acba9e37a" />

19. Run a command to upload 500 1kb files to S3 using 50 threads, the operation took 22.9 seconds to complete:
time parallel --will-cite -a object_ids -j 50 aws s3 cp 1KB.file s3://${bucket}/run3/{}  

<img width="128" alt="19" src="https://github.com/user-attachments/assets/1ca25940-aea3-4a1b-a824-11ebf2388c5d" />

20. Run a command to upload 500 1kb files to S3 using 50 threads, the operation took 8 seconds to complete:

<img width="872" alt="20" src="https://github.com/user-attachments/assets/11f59341-c876-4676-9b72-49f4d5589d7c" />

21. Run a command to download a 1GB file from an S3 bucket and upload the file into the same bucket with a different prefix, the operation took 22 seconds to complete:

<img width="886" alt="21" src="https://github.com/user-attachments/assets/b0f6306f-744a-4e0f-a112-803b0569e8a9" />

22. Run a command to use PUT Copy (copy-object) to move the file, the operation took 3 minutes 31 seconds to complete. 

<img width="934" alt="22" src="https://github.com/user-attachments/assets/ff8c48e3-90ba-4770-81bb-ce3fc7424109" />

### EFS Storage performance

1. Connect to the SID-performane instance using EC2 Instance connect. Run a command to generate 1024 zero byte files, the operation takes 8 seconds to complete.

<img width="548" alt="1" src="https://github.com/user-attachments/assets/506c8281-828e-433f-a7ea-4b4dc922e1a3" />

2. Run a command to generate 1024 zero byte files using multiple threads, the operation took 4 seconds to complete.

<img width="862" alt="2" src="https://github.com/user-attachments/assets/65c0c4ec-079f-4ed1-aebc-f6d8615fb8ac" />

3. Run a command to generate 1024 zero byte files in multiple directories using multiple threads, the operation took 0.4 seconds to complete

<img width="884" alt="3" src="https://github.com/user-attachments/assets/77ed7cca-1f2a-4444-8e2b-96c60acd8927" />

4. Run a command to write a 2GB file to EFS using 1MB block size and sync once after each file, the operation took 6 seconds to complete

<img width="712" alt="4" src="https://github.com/user-attachments/assets/2453f99b-b41f-4019-9c39-3a0a5bb16a31" />

5. Run a command to write a 2GB file to EFS using 16MB block size and sync once after each file, the operation took 7 seconds to complete

<img width="713" alt="5" src="https://github.com/user-attachments/assets/26154774-5b2c-438e-b0d8-4daad56b7441" />

6. Run a command to write a 2GB file to EFS using 1MB block size and sync after each block, the operation took 51 seconds to complete

<img width="713" alt="6" src="https://github.com/user-attachments/assets/775c9c85-84da-496b-8b8f-2d5af14fca9d" />

7. Run a command to write a 2GB file to EFS using 16MB block size and sync once after each block, the operation took 14 seconds to complete

<img width="716" alt="7" src="https://github.com/user-attachments/assets/3e1beeab-ac79-4821-9a02-065b60546a7f" />

8. Run a command to write a 2GB of data to EFS using 1MB block size and 4 threads, the operation took 13 seconds to complete

<img width="594" alt="8" src="https://github.com/user-attachments/assets/c40adfb0-c69a-48fb-91e6-f06455c8d086" />

9. Run a command to write a 2GB of data to EFS using 1MB block size and 16 threads, the operation took 5 seconds to complete:
time seq 0 15 | parallel --will-cite -j 16 dd if=/dev/zero \
of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N)-{} bs=1M count=128 oflag=sync

<img width="113" alt="9" src="https://github.com/user-attachments/assets/fb5708eb-adc7-498b-ac12-d8341b485750" />

10. Clean up resources

10.1 Delete the CloudFormation stacks

## Migrating data to AWS
In the hands on labs, I will be doing the following: 
* Use DataSync to migrate data from on-premises (simulation) to AWS
* Use Storage gateway to present the file system back on premises

1. Launch 2 CloudFormation stacks, one stack will be launched in us-west-2 (this is the on-premises environment) and another stack should be launched in us-east=1 (cloud)

<img width="955" alt="1" src="https://github.com/user-attachments/assets/1453896c-a090-419e-9502-00ba10955ad5" />

<img width="952" alt="4" src="https://github.com/user-attachments/assets/ae224ac2-ff08-43cc-b3b3-3c7dfef6f397" />

2. Connect to the SID-appserver instance using a EC2 Instance connect (SSH-session) and run a command to mount the NFS export

<img width="568" alt="5" src="https://github.com/user-attachments/assets/a1500cfd-9bd1-4697-8f9f-ea7b7ba3f25c" />

3. Run a command to verify that there are 200 images in the folder

<img width="881" alt="6" src="https://github.com/user-attachments/assets/c6ca0414-b620-45b4-a4c3-425d37f5794a" />

4. Activate the DataSync agent

<img width="958" alt="8" src="https://github.com/user-attachments/assets/5a74bd87-f2e4-47fb-bec2-c771f8ff9665" />

5. Create a NFS location

<img width="952" alt="9" src="https://github.com/user-attachments/assets/d3648cd5-8fe8-4478-9dcc-758abd73cf49" />

6. Create a S3 location

<img width="956" alt="10" src="https://github.com/user-attachments/assets/fd21e217-87ed-470a-9856-a02148ddbf86" />

7. Create a DataSync task

<img width="952" alt="11" src="https://github.com/user-attachments/assets/7a7e244e-f006-40b4-8a0a-677d04cf7c1d" />

8. Run the DataSync task, the task is tranferring the 200 files from (on premises) NFS to S3. 

<img width="953" alt="12" src="https://github.com/user-attachments/assets/8872aa98-72bc-44c0-90d8-0716f7027059" />

9. The S3 bucket now contains the 200 files that we transferred

<img width="953" alt="13" src="https://github.com/user-attachments/assets/49493609-8e99-41b1-a3bf-a293e4ae9c5e" />

10. Activate a File Gateway 

<img width="946" alt="14" src="https://github.com/user-attachments/assets/1503c1c0-1912-4682-983a-ad3e063f8612" />

<img width="953" alt="15" src="https://github.com/user-attachments/assets/c5e798d5-4972-4a66-99c1-a3015057d640" />

11. Create an NFS share on the File Gateway

<img width="956" alt="16" src="https://github.com/user-attachments/assets/3dc5efc2-6630-408f-b191-dc2d230e1775" />

12. Connect to the SID-appserver instance using a EC2 Instance connect (SSH-session) and run the following command

<img width="342" alt="17" src="https://github.com/user-attachments/assets/6df9e8e1-508a-42c9-9db3-b8edbf79fa19" />

13. Run a command to mount the NFS share on the application server

<img width="911" alt="18" src="https://github.com/user-attachments/assets/d706212f-bf64-402a-87d7-e18625de7d8b" />

14. Run a command to verify that the same set of files exist on both NFS shares

<img width="389" alt="19" src="https://github.com/user-attachments/assets/c7df7460-00ed-41c7-89df-b2a48532dbfb" />

15. Run a command to create a new file on the NFS server

<img width="600" alt="20" src="https://github.com/user-attachments/assets/7a3394fa-0a9f-4cab-9312-660b8d786eee" />

16. Copy the new file to the S3 bucket by re-running the DataSync task

<img width="954" alt="21" src="https://github.com/user-attachments/assets/94e6b5d0-0207-47ec-b51a-3107baa75dff" />

17. Validating that the data transfer to S3 was successful

<img width="955" alt="22" src="https://github.com/user-attachments/assets/77289fdb-8d56-41ea-8b57-572d77841bfd" />

18. Run a command in the ssh-session to validate that the file exists locally 

<img width="290" alt="23" src="https://github.com/user-attachments/assets/f5a66cc3-a0b2-4c48-8f8d-6a976bc7c57e" />

19. Run a command in the ssh-session to validate that the file exists in the file gateway. The file was written to the S3 bucket via DataSync, not through the File Gateway share. 

<img width="284" alt="24" src="https://github.com/user-attachments/assets/b9cc556e-66c8-487a-b5ab-01e670b403c1" />

20. Refresh the metadata cache on File gateway to reflect the change

<img width="955" alt="25" src="https://github.com/user-attachments/assets/6e6ab1c0-0bc9-49cc-b841-112233ccaf14" />

21. In the SSH session, run a command to validate if the image exists in the file gateway. The image is now visible.

<img width="283" alt="26" src="https://github.com/user-attachments/assets/a1659340-2c94-462e-9004-0634e9e3bdab" />

22. Run the following command to unmount the file share: sudo umount /mnt/data

23. In the us-east-1 region, delete the Task, Locations and Agent that were previously created

24. In us-west-2 region, run a command to create a new file in the S3 bucket through File Gateway

<img width="587" alt="27" src="https://github.com/user-attachments/assets/381bc505-9e13-43ec-ac74-0d69478a6e7b" />

25. In the S3 bucket, the new file that we just created is visible

<img width="950" alt="28" src="https://github.com/user-attachments/assets/9275d5a3-a372-4369-a454-c99a51d218dd" />

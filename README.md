# s3-testing

Abusing S3 Bucket Permissions

IDENTIFYING S3 BUCKETS
There Are various ways to identify S3 buckets few of which are

HTML inspection (S3 Bucket URL hardcoded in HTML of webpage)
Brute-Force
Google Dorking
DNS caching
Reverse IP lookup (Bing Reverse IP)
Tools on GitHub
TESTING AND ABUSING S3 BUCKET PERMISSIONS
Once you have identified S3 buckets, it is time to test out permission they have and once they are identified, we can start abusing them. But what are the permissions in particular assigned to S3 buckets? Let us have a brief look at them

READ PERMISSION
This permission allows reading the contents/objects of the bucket. At the object level, it allows reading the contents as well as metadata of the object. Testing this permission is really simple. The first way is to try and access it via URL. Use this as a skeleton and modify as according to your needs

http://[name_of_bucket].s3.amazonaws.com

This same thing can be done by using the AWS command line as well. Following is the command that can be used to list the contents

aws s3 ls s3://[name_of_bucket]  --no-sign-request
No sign request switch that we used in the above command is used to specify not to use any credential to sign the request. A bucket that allows reading its contents will spit out all the contents once requested by any of the above methods. HTML request will show output on the XML page and the command line will give a list of files.

WRITE PERMISSION
This permission allows to create, overwrite and delete objects in a bucket. Speaking of object-level access, it allows editing of the object itself. As far as testing goes, this permission can only be tested by the AWS command-line interface, unlike reading permission where you had a chance to test it through a web browser. Following is the command that you can use and modify according to your needs

aws s3 cp localfile s3://[name_of_bucket]/test_file.txt –-no-sign-request
A bucket that allows unrestricted file upload will show a message that the file has been uploaded.

READ_ACP
At bucket, level allows to read the bucket’s Access Control List and at the object level allows to read the object’s Access Control List. Read ACL can be tested with the AWS command line using the following command

aws s3api get-bucket-acl --bucket [bucketname] --no-sign
aws s3api get-object-acl --bucket [bucketname] --key index.html --no-sign-request
Once we execute both these commands, they will output a JSON describing the ACL policies for the resource being specified.

WRITE_ACP
This policy at the bucket level allows setting access control list for the bucket. At the object level, allows setting access control list for the objects. To test this, all you need to do is run the following g command :

aws s3api put-bucket-acl --bucket [bucketname] [ACLPERMISSIONS] --no-sign-request
To test for a single object, following command can be used

aws s3api put-object-acl --bucket [bucketname] --key file.txt [ACLPERMISSIONS] --no-sign-request
For this though, there is no output shown in the form of a message

FULL_CONTROL
This term is self explanatory which means it grants full control (READ,WRITE,READ_ACP, WRITE_ACP). The same goes for object-level access as well.

Finally is the most destructive one which is any authenticated AWS CLIENT. This permission allows any authenticated AWS client to access the account regardless of what type of account they are using. To test this, simply take any command from above and replace –no-sign-request with –profile [your_profile_name].

RESOURCE TO PRACTICE
Following is an excellent open-source resource where you can test out all the permissions. Assuming that ACL and proper policies are in place, only one or few commands might show successful output. You can always set up your own S3 bucket and test out these commands as an unauthenticated user on your bucket. You can then see the output. Later you can add on policies and Access control measures on your bucket and test again with these commands.

This website gives CTF like challenges where the content of each flag is entry to the next challenge so you can test out your skills in securing the cloud with these challenges.


To demonstrate the above commands, we have created an AWS bucket and configured it in a vulnerable fashion. Let us look at the above commands one by one. First, we will show you what the output will look like once we run the above commands and later, we are going to look at the vulnerable policy that we have set on our bucket and also other misconfigured buckets that might have on open public internet.

READ PERMISSION

WRITE PERMISSION
In this, when we ran command for first time, it showed error because s3 bucket blocked file uploads. Later on, we modified permission hence it allowed file uploads.


READ_ACP

WRITE_ACP
You can test out write ACP by creating your own custom Access control Policy and uploading it by running command mentioned previously.

VULNERABLE POLICY
Let us have a look at vulnerable policy we set on our bucket.

{
     "Version": "2012-10-17",
     "Statement": [
         {
             "Sid": "Statement1",
             "Effect": "Allow",
             "Principal": {
                 "AWS": ""
             },
             "Action": "",
             "Resource": [
                 "arn:aws:s3:::temp4blog",
                 "arn:aws:s3:::temp4blog/*"
             ]
         }
     ]
 }
The main problem here is with the Action tag which is set to *. It means that to allow all the actions including reading write upload delete etc. This is a quick and convenient way for administrators to assign access control to the bucket but at the cost of convenience comes the security which leaves the bucket vulnerable. To mitigate this, administrators should set each and every permission/Action manually as per requirement and configure the bucket properly such that public access is blocked.

Here are summed up steps you can take to secure an s3 bucket or recommendations you can give after you have found the s3 bucket.

Making Bucket private / Disallow public access
Configuring S3 bucket policies properly
Using S3 ACLs
Using assumed IAM roles which has access to the bucket for read/write
Using Presigned URLs
Enabling CloudTrail and Server access logging for your bucket.

src: yeswehack.com/learn-bug-bounty/abusing-s3-bucket-permissions

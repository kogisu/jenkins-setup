# static
Cloud devops part 3 (CI/CD) Jenkins setup on EC2

## Part 1: Create user account
Using the aws console, go to IAM (Identity and Access Management).  
Create new group called `jenkins`.  Provide the following policies
* AmazonEC2FullAccess
* AmazonVPCFullAccess
* AmazonS3FullAccess

Create new user `jenkins` under Users in IAM and add access types
* Programmatic access
* AWS Management Console access

Add user `jenkins` to group `jenkins`.  Skip tags.  Capture the `Access Key`, `Secret Access Key` and `password` for future login.  
**Note:** that you should also capture your account # to login
Sign in to newly created user account by logging out of aws console, entering the account #, and captured details from step above.  You can also login using the url..
```
https://your_aws_account_id.signin.aws.amazon.com/console/
```

Launch EC2 Instance using the `Ubuntu 18.04 LTS amd64` AMI (amazon machine image), create new key and save pem.  

Create Security Group for EC2  
Under `Network and Security` in EC2, select `Security Groups`.  
Use name `jenkins`, and add the following rules:
* Custom TCP Rule, Protocol: TCP, port range: 8080, source: 0.0.0.0/0
* SSH rule, protocol: SSH, port range: 22, source: "My IP" <-- this provides extra security by limiting access to you

SSH into instance  
To ssh into the newly created instance...  
```
chmod 400 "/absolute/path/to/.pem" <-- provides rw access to owner
ssh -i "/absolute/path/to/.pem" ubuntu@<public_ip_of_instance>
```

## Part 2: Install Jenkins on Ubuntu
* `apt update`
* `apt upgrade`
* `apt install default-jdk`

Get repo using `wget`
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
```

Append debian package repo address to the server's sources.list
```
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```

Update so that `apt` will use the new repo:  
```
sudo apt update
```

Install jenkins and its dependencies:  
```
sudo apt install jenkins
```

Confirm that jenkins is running using `systemctl`  
```
sudo systemctl status jenkins
```

## Part 3: Set up Jenkins  
If everything above ran smoothly, you should be able to access jenkins at   
`http://server_ip_or_domain:8080`   

To unlock jenkins, `cat` initial admin password
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Create new password upon logging in to make the login process easier  

## Part 4: Install Blue Ocean plugins
Go to The left-hand bar and select `Manage Jenkins` -> `Manage Plugins` -> Available tab.  
Then install the plugins for blue ocean by searching `blue ocean`
* Blue Ocean
* Config API for Blue Ocean
* Events API for Blue Ocean
* Git Pipeline for Blue Ocean
* GitHub Pipeline for Blue Ocean
* Pipeline implementration for Blue Ocean
* Blue Ocean Pipeline Editor
* Display URL for Blue Ocean
* Blue Ocean Executor info

**Install without restart**

Restart after install.  If jenkins does not restart automatically after install, restart via cli.  To restart jenkins from the cli...          
```
sudo systemctl restart jenkins
```

Verify that installed plugins work by going to `Open Blue Ocean` on the jenkins home page.

## Part 5: Create github authentication token
From github home page, go to settings -> Developer settings -> personal access tokens  
Create new access token with name `jenkins` and provide following scopes
* repo: (full control)
* admin:repo_hook (full control)
* user:
  * read:user
  * user:email

**Copy access key**   

## Part 6: Create new pipeline
1. Go to the Blue Ocean console in Jenkins and select `Create new pipeline` and `Github`.  
2. Enter the github access token previously obtained and connect.  
3. Select your account, `static` repo.  Pipeline should automatically run.

## Part 7: Install AWS Plugins
Go to The left-hand bar and select `Manage Jenkins` -> `Manage Plugins` -> Available tab.
Then install the plugins for aws by searching `aws`  
* Pipeline: AWS Steps

**Install without restart**

Restart after install.  If jenkins does not restart automatically after install, restart via cli.  To restart jenkins from the cli...
```
sudo systemctl restart jenkins
```

## Part 8: Add credentials 
1. Click 'Credentials' from the sidebar.
2. Click on "(global)" from the list, and then "Add credentials" from the sidebar.
3. Choose "AWS Credentials" from the dropdown, add "aws-static" on ID, add a description like "Static HTML publisher in AWS," and fill in the AWS Key and Secret Access Key generated when the IAM role was created.  **Note:** The aws credentials ID is `aws-static`.
4. Click OK, and the credentials should now be available for the rest of the system.

## Part 9: Set up S3 Bucket
1. Login to console for the `jenkins` user and select `S3`
2. Create new bucket with unique name
3. Take note of region
4. Skip until permissions
5. For `Set Permissions`, uncheck `"Block all public access"`.  Review and continue.
6. Click the bucket to get to the configuration panel
7. Select `Properties` tab, click `"Static website hosting"`.  Enable `Use this bucket to host a website` and type `index.html` for the html doc.  Click `Save`.
8. Select the `permissions` tab
9. Click the `Bucket policy` and add the following
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3::BUCKET_NAME/*"
    }
  ]
}
```
10. Save, and `Permissions public` should now show in tab.  

## Part 10:
To add functionality for aws, see documentation for [pipeline-aws-plugin](https://github.com/jenkinsci/pipeline-aws-plugin#withAWS).

To link to aws, use `withAWS` to provide authorization for steps. 

For example, to upload to s3, set up stage as following...
```
stage('Upload to AWS') {
    steps {
        withAWS(region: 'us-east-1', credentials: 'aws-static') {
            s3Upload(file: 'index.html', bucket: 'static-site-jenkins', path: 'index.html')
        }
    }
} 
```

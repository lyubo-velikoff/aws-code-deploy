# AWS Code Deploy example

This is an example of how to set up Github repository with AWS Code deploy service. It can be combined along with AWS Pipeline which automates the whole process.

## Requirements
1) abbspec.yml file

*Example of appspec.yml*

```yml
version: 0.0
os: linux
files:
   - source: /
     destination: /var/www/html
permissions:
   - object: /var/www/html
     pattern: "**"
     owner: ec2-user
     group: apache
     mode: 755
     type:
       - file
hooks:
   BeforeInstall:
    - location: scripts/installapache.sh
      runas: root
    - location: scripts/startapache.sh
      runas: root
   AfterInstall:
    - location: scripts/restartapache.sh
      runas: root
```

## AWS code deploy example.

<b>1. Create custom Code Deploy AIM Role</b>

*Note: You will have to remove the auto generated policy and add an inline one.*

*Copy the policy role*

```json
// Policy Role for Code Deploy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "autoscaling:PutLifecycleHook",
        "autoscaling:DeleteLifecycleHook",
        "autoscaling:RecordLifecycleActionHeartbeat",
        "autoscaling:CompleteLifecycleAction",
        "autoscaling:DescribeAutoscalingGroups",
        "autoscaling:PutInstanceInStandby",
        "autoscaling:PutInstanceInService",
        "ec2:Describe*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

*Edit the truested rule and copy and paste the below code*

```json
// Policy Trust for Code Deploy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "codedeploy.us-west-2.amazonaws.com",
          "codedeploy.us-east-1.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

<b>2. Create custom EC2 Instance Code Deploy AIM Role</b>

```json
// Instance Role for EC2 Instance
{ 
    "Version": "2012-10-17", 
    "Statement": [   
      {     
          "Action": [       
              "s3:Get*",       
              "s3:List*"     
          ],     
          "Effect": "Allow",     
          "Resource": "*"   
      } 
    ]
}
```

<b>3. Create and launch an EC2 instance with the EC2 Instance Code DEploy AIM Role</b>

<b>4. Once instance is ready, SSH connect to it and enter the following</b>

```bash
yum -y update

yum install -y aws-cli

cd /home/ec2-user
```

<b>5. Set up AWS Cli</b>

```bash
aws configure 
```

*Note: It will ask you for AWS Key(deploy-cli), Secret credentials, region(eu-west-2) and default format (json). The KEY and Secret can be found under users.*

<b>6. Copy Code deploy agent settings</b>

```bash
aws s3 cp s3://aws-codedeploy-eu-west-2/latest/install . --region eu-west-2
```

for ubuntu 14.04:

```bash
sudo apt-get install ruby2.0
```

for ubuntu 16.04

```bash
sudo apt-get install ruby
```

```bash
wget https://aws-codedeploy-eu-west-2.s3.amazonaws.com/latest/install
```

<b>7. Give agent settings permission</b>

```bash
chmod +x ./install
```
<b>8. This is simply a quick hack to get the agent running faster (Optional)</b>

```bash
sed -i "s/sleep(.*)/sleep(10)/" install 
```

<b>9. Install Code Deploy agent from region settings</b>

```bash
./install auto
```

<b>10. Verify if code deploy agent is running</b>

```bash
service codedeploy-agent status 
```

*Example output: The AWS CodeDeploy agent is running as PID 8190*

## Set up apache user and give permissions to read write

*Note: Follow AWS ec2 instance configuration tutorial*

## Set up Code Deploy application - follow AWS CodeDeploy documentation

## Set up Code Pipelines application - follow AWS CodePipelines documentation

AWS
===

# Configure

```shell
$ aws configure
```

# EC2

List Key Pairs

```shell
aws ec2 describe-key-pairs
```

Describe image

```shell
aws ec2 describe-images --image-ids ${imageId}
```

Filter Images (Examples)

```shell
aws ec2 describe-images --owners ${imageOwnerId} --filters "Name=architecture,Values=x86_64" "Name=root-device-type,Values=ebs" "Name=name,Values=*ubuntu-xenial*" "Name=virtualization-type,Values=hvm"

aws ec2 describe-images --owners ${imageOwnerId} --filters "Name=architecture,Values=x86_64" "Name=root-device-type,Values=ebs" "Name=name,Values=*ubuntu-xenial*" "Name=virtualization-type,Values=hvm" --query "Images[*].{ID: ImageId, Desc:Description, CreationDate:CreationDate}" --output text

aws ec2 describe-images --owners ${imageOwnerId} --filters "Name=architecture,Values=x86_64" "Name=root-device-type,Values=ebs" "Name=name,Values=*ubuntu-xenial*" "Name=virtualization-type,Values=hvm" "Name=block-device-mapping.volume-type,Values=gp2" "Name=hypervisor,Values=xen" --query "Images[*].{ID: ImageId, Desc:Description, CreationDate:CreationDate}" --output text | grep -v UNSUPPORTED
```

Run Instance

```shell
aws ec2 run-instances --image-id ${imageId} --security-group-ids ${groupId} --count 1 --instance-type t2.micro --key-name ${keyName} --query 'Instances[0].InstanceId'
```

Run instance within a subnet

```shell
aws ec2 run-instances --image-id ${imageId} --instance-type t2.micro --key-name ${keyName} --security-group-ids ${groupId1} ${groupId2} --count 1 --subnet-id ${subnetId}
```

Instance Details

```shell
aws ec2 describe-instances --instance-ids ${instanceId}
```

Create tag for instance

```shell
aws ec2 create-tags --resources ${instanceId} --tags Key=service,Value=webapp Key=environment,Value=production
```

Reboot instance

```shell
aws ec2 reboot-instances --instance-ids ${instanceId}
```
## VPC/Subnets


List all vpcs

```shell
aws ec2 describe-vpcs
```

Create vpc with cidr

```shell
aws ec2 create-vpc --cidr-block 172.32.0.0/16
```

Enable DNS/Hostname support for given vpc

```shell 
aws ec2 modify-vpc-attribute --vpc-id ${vpcId} --enable-dns-support "{\"Value\":true}"
aws ec2 modify-vpc-attribute --vpc-id ${vpcId} --enable-dns-hostnames "{\"Value\":true}"
```

List all subnets

```shell
aws ec2 describe-subnets
```

List all subnets for specific vpc

```shell
aws ec2 describe-subnets --filters Name=vpc-id,Values=${vpcId}
```

Create subnets within vpc

```shell
aws ec2 create-subnet --vpc-id ${vpcId} --cidr-block 172.32.0.0/24
aws ec2 create-subnet --vpc-id ${vpcId} --cidr-block 172.32.1.0/24
```

Create subnets with different availability zones

```shell
aws ec2 create-subnet --vpc-id ${vpcId} --cidr-block 172.32.2.0/24 --availability-zone "us-west-1c"
```

*Different availability zone is required for db instance*


Create and attach internet gateway to vpc

```shell
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id ${vpcId} --internet-gateway-id ${internetGatewayId}
```

*First command creates internet gateway and return gateway id*

Create route table for specific vpc

```shell
aws ec2 create-route-table --vpc-id ${vpcId}
```

Create route and attach it to route table and internet gateway

```shell
aws ec2 create-route --route-table-id ${routeTableId} --destination-cidr-block 0.0.0.0/0 --gateway-id ${internetGatewayId}
```


Attach route table with subnet

```shell
aws ec2 associate-route-table  --subnet-id ${subnetId} --route-table-id ${routeTableId}
```

Attach public IP (when launching instance) with subnet

```shell
aws ec2 modify-subnet-attribute --subnet-id ${subnetId} --map-public-ip-on-launch
```
### process

1. create vpc

	a. enable dns support

	b. enable dns hostnames
2. create subnets ( private/public ) for vpc
3. create internet gateway
4. attach internet gateway to vpc
5. create route table for vpc
6. associate route table to subnet ( Public )
7. modify subnet for auto assign public ip
8. create route to route table and gateway
9. create security groups ( ssh/db/http ) for vpc
10. authorize groups
11. create keypair
12. launch ec2 instance with keypair and subnet

13. create rds db subnet group
14. create db instance with db-subnet-group and vpc-security-group
## Security Groups

Create Security Group

```shell
aws ec2 create-security-group --group-name web-sg --description "security group for my application"
```

Allow Web access to everyone

```shell
aws ec2 authorize-security-group-ingress --group-name web-sg --protocol tcp --port 80 --cidr 0.0.0.0/0
```

Create SSH security group

```shell
aws ec2 create-security-group --group-name webSSHGrp --description "Security group for SSH access" --vpc-id ${vpcId}
```

Allow SSH access to SSH security group

```shell
aws ec2 authorize-security-group-ingress --group-id ${groupId} --protocol tcp --port 22 --cidr 0.0.0.0/0
```

Create DB security group

```shell
aws ec2 create-security-group --group-name webDBGrp --description "Security group for DB access" --vpc-id ${vpcId}
```

Allow DB access to specific security group only

```shell
aws ec2 authorize-security-group-ingress --group-id ${groupId} --source-group ${sourceGroupId} --protocol tcp --port 5432
```

*This will restrict connections to source security groups only.*


List security groups

```shell
aws ec2 describe-security-groups
```

List security groups for specific vpc

```shell
aws ec2 describe-security-groups --filters "Name=vpc-id,Values=${vpcId}"
```


Create Key-Pair

```shell
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > ~/.ssh/my-key.pem
```

## Querying

List subnet with Id and cidr

```shell
aws ec2 describe-subnets --filters "Name=vpc-id,Values=${vpcId}" --query 'Subnets[*].{ID:SubnetId,CIDR:CidrBlock}'
```

Describe route table

```shell
aws ec2 describe-route-tables --route-table-id ${routeTableId}
```

Get Public IP of instance

```shell
aws ec2 describe-instances --instance-ids ${instanceId} --query 'Reservations[0].Instances[0].PublicIpAddress'
```

Filter instance by tags

```shell
aws ec2 describe-instances --filter "Name=tag:Name,Values=myapp"
```

	aws rds describe-db-subnet-groups

	aws rds describe-db-security-groups

	aws rds create-db-security-group --db-security-group-name sg-app-db --db-security-group-description "security group for application db"
# RDS

List instances

```shell
aws rds describe-db-instances
```

List database engines

```shell
aws rds describe-db-engine-versions --engine postgres
```

List engine instance class

```shell
aws rds describe-orderable-db-instance-options  --engine postgres --engine-version 9.6.3 --query 'OrderableDBInstanceOptions[*].DBInstanceClass' --output text
```

Create instance

```shell
aws rds create-db-instance --db-name postgresql --db-instance-identifier my-db-pgsql --allocated-storage 5 --db-instance-class db.t2.small --engine postgres --master-username ${userName} --master-user-password ${myPassword} --vpc-security-group-ids ${securityGroupId} --db-subnet-group-name ${subnetGroupName}
```

Delete instance

```shell
aws rds delete-db-instance --skip-final-snapshot --db-instance-identifier mysql-small-db
```
## Subnets

Create db Subnet

```shell
aws rds create-db-subnet-group --db-subnet-group-name myapp-db-subnet --db-subnet-group-description "postgresql db subnet group" --subnet-ids ${subnetId1} ${subnetId2}
```

*Atleast two subnets are required with different availability zones*

List db subnet groups

```shell
aws rds describe-db-subnet-groups
```
## Security Groups

Create DB Security group

```shell
aws rds create-db-security-group --db-security-group-name sg-app-db --db-security-group-description "security group for application db"
```

List DB Security groups

```shell
aws rds describe-db-security-groups
```
# IAM

List users

```shell
aws iam list-users
```

Create user

```shell
aws iam create-user --user-name ${userName} --path /myblog
```

Give programmetic access to user

```shell
aws iam create-access-key --user-name ${userName}
```

List groups

```shell
aws iam list-groups
```

Create S3 Policy

```shell
aws iam create-policy --policy-name ${policyName} --policy-document '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteObject",
        "s3:Put*",
        "s3:Get*",
        "s3:List*"
      ],
      "Resource": [
        "arn:aws:s3:::*"
      ]
    }
  ]
}'
```


Attach policy to user

```shell
aws iam attach-user-policy --user-name ${userName} --policy-arn ${policyArn}
```

List access keys

```shell
aws iam list-access-keys --user-name ${userName}
```

List Roles

```shell
aws iam list-roles
```
# S3

Create bucket

```shell
aws s3api create-bucket --bucket ${bucketName} --create-bucket-configuration LocationConstraint=us-west-1
```

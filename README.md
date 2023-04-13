# Service Catalog Terraform Reference Architecture

## Solution Overview

**The Service Catalog team recently introduced support for [Terraform open source](https://aws.amazon.com/about-aws/whats-new/2023/04/aws-service-catalog-terraform-open-source/) so we reccomend users to use this instead. This Sample will be deprecated in the future. More details on Service Catalog and Terraform open source can be found in the [documentation](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-Terraform.html).**

---


**AWS Service Catalog and Terraform Terminology**  
- hub or fulfillment account: The account where the Terraform server or engine will live. The terraform instance will receive 
a template path and parameters from the custom lambda which will then be executed by the Terraform binary.
- spoke account: The spoke accounts that will send Terraform file paths and parameters to the Terraform engine in the Hub account for execution.

This solution requires that a Terraform server be deployed and available. It is possible to use an existing Terraform server though instructions are not included here.  
To demonstrate the process we have supplied a standardized CloudFormation templates which 
creates a new Amazon Elastic Compute Cloud (Amazon EC2) instance and installs Terraform. 

You can use one Terraform server to target resources across multiple AWS accounts. 
For the Terraform server to operate, you must create AWS Identity and Access Management (IAM) 
roles in each of the spoke accounts and create an AWS Lambda launch function in each desired region of each spoke account. 
For example, a deployment for 3 accounts that each use us-east-1 and us-east-2 regions will result in 1 Terraform server account, 
3 spoke account roles, and 6 Lambda launch functions. The hub account and region containing the Terraform server may also be a spoke 
account, in which case you must also deploy the IAM role and Lambda launch functions in the hub account and region.

The key to the solution is a CloudFormation template and custom resource lambda that act as a proxy to Terraform. 
The CloudFormation template uses a custom resource, implemented via an AWS Lambda function to relay template paths and parameters to the Terraform binary on the EC2 instance. 
The CloudFormation template can then be customized to capture specific parameters and used as a resource in an AWS Service Catalog product. 


![Solution Overview](documentation/images/ServicsingelerraformLab.png)  


The Terraform files are placed in an Amazon Simple Storage Service (Amazon S3) bucket that can be accessed from AWS Service Catalog 
in each of the spoke accounts. See the following figure.  

![Solution Overview](documentation/images/ServicCatalogandTerr.png)  


---
## Installation  

Copy the Service Catalog Terraform components to a S3 bucket to be launched from CloudFormation.  
It is assumed that the user has Administrative privileges in IAM for S3, CloudFormation, EC2, VPC, Lambda, and Service Catalog.

- Clone the repository to your local computer

``` 
  git clone https://github.com/aws-samples/aws-service-catalog-terraform-reference-architecture.git
  
```
The **aws-service-catalog-terraform-reference-architecture** folder is created and should look like this

```
  cd  aws-service-catalog-terraform-reference-architecture
  ls -l

    CODE_OF_CONDUCT.md
    CONTRIBUTING.md
    documentation
    LICENSE
    NOTICE
    README.md
    ServiceCatalogSamples
    TerraformCustomResourceHandler
    TerraformScripts

```

Replace your AWS account ID with the place holder in the lines below **[YOUR-ACCOUNT-ID]** NO DASHES

AWS CLI Install [Instructions](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

1.  Create a S3 bucket to host the installation content  
``` aws s3 mb s3://scterraform-[YOUR-ACCOUNT-ID] ```  

2.  Copy the following files to the bucket maintaining the file structure:
    ```
     aws s3 sync ./TerraformCustomResourceHandler/bin/ s3://scterraform-[YOUR-ACCOUNT-ID]/TerraformCustomResourceHandler/bin/  
     aws s3 sync ./ServiceCatalogSamples/ s3://scterraform-[YOUR-ACCOUNT-ID]/ServiceCatalogSamples/  
     aws s3 sync ./TerraformScripts/ s3://scterraform-[YOUR-ACCOUNT-ID]/TerraformScripts/  
    ```  
Login to S3 console and make the folder TerraformScripts public so that the templates can be accessed from child accounts.

You now have all the files needed for launch in the S3 Bucket which has the same file structure as this github repo:  

    S3 Bukcet: scterraform-[YOUR-ACCOUNT-ID]  
    ├── TerraformScripts/  
    │   ├── cloudformation-templates/  
    │   │   ├── terraform-architecture-single-account.yaml  
    │   │   ├── terraform-fulfillment-server.yaml  
    │   │   ├── terraform-launch-lambda.yaml  
    │   │   └── terraform-spoke-principals.yaml  
    │   └── bin/  
    │       └── sc_terraform_wrapper-1.2-py3-none-any.whl  
    └── TerraformCustomResourceHandler/  
        └── bin/  
            └── aws-servicecatalog-terraform-wrapper.jar  

**Note** We have only listed the files you need for this demonstration, more files are included in the actual git repository

---

 ### Installing the Service Catalog Terraform reference architecture into a single account hub account  
 
  1. Navigate to the S3 [console](https://console.aws.amazon.com/s3/).In the S3 Console, choose the **TerraformScripts/cloudformation-templates/terraform-architecture-single-account.yaml** file
  2. Copy the **Object URL**
  2. Navigate to the CloudFormation [console](https://console.aws.amazon.com/cloudformation/).
  3. Verify the region.
  4. Choose **Create Stack**.
  5. Under Choose a template, select Specify an **Amazon S3 template URL**.
  6. Paste the URL you copied from step 1 above  
  7. Choose **Next**.
  8. For Stack name, type **TerraformArchitecture-SingleAccount**.
  9. Leave all the parameters as defaults.
  10. Choose **Next**
  11. On the **Configure stack options** page chose **Next**
  12. On the **Review** page
  - choose the check box for **I acknowledge that AWS CloudFormation might create IAM resources with custom names.**
  - choose the check box for **I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND.** 
  14. choose **Create Stack**
  

  When the Status changes to **CREATE_COMPLETE** once the stack is created. 
  The stack output will look similar to the image below.
![alt text](documentation/images/oneinstall004.png)


## Create AWS Service Catalog portfolio and product based on Terraform
 
1. Navigate to the S3 [S3 console](https://console.aws.amazon.com/s3/).
2. Choose the __**terraformarchitecture-single-terraformconfigstore-xxxx **__ bucket
3. Choose **Upload**
4. Use the file explorer opened earlier to open the **ServiceCatalogSamples** folder
5. Select and drag over the following files
- sc-sample-lamp.json
- sc-sample-lamp.tf
- sc-sample-port-product-setup.json
- sc-sample-S3.json
- sc-sample-S3.tf
6. Choose **Upload**. 
7. In the S3 Console, choose the **sc-sample-port-product-setup.json** file
8. Click on it and copy the Object URL
6.  Navigate to the **CloudFormation console** https://console.aws.amazon.com/cloudformation/
7.  Verify the **region**. 
8.  Choose **Create Stack**.
9.  Under Choose a template, select Specify an Amazon S3 template URL.
10.  Paste the URL you copied in the step above. 
12.  For Stack name, type **SCTFportfoliosetup**.
13.  For all the other parameters use the defaults.
14. Choose **Next**.
15. Choose **Create Stack** to create the Service Catalog Sample Terraform Portfolio.
16. When the stack is complete, choose the **Outputs** tab
17. Choose the **NewPortFolio** to complete the setup and launch the sample products.

![alt text](documentation/images/portsetup001.png)
![alt text](documentation/images/portsetup002.png)
![alt text](documentation/images/portsetup003.png)

Service Catalog Console
![alt text](documentation/images/portsetup004.png)

### Congratulations, You have completed setting up the Service Catalog Terraform Reference Architecture components in a single account.


---
## Set up the spoke accounts

  For each spoke account, you’ll need to enable execution from the hub account by creating an executing role with a 
  trust relationship back to the hub account. Then, create a Lambda launch function in each desired region within the spoke account. 
  Finally, add each spoke account to the SNS topic that queues requests for the spoke account. 
  The supplied CloudFormation stacks will provision these roles and Lambda functions.
  
### Create the spoke account Terraform roles

**Note:** Make sure the **TerraformArchitecture-SingleAccount** CloudFormation stack has a status of CREATE_COMPLETE before proceeding.   

1.  Sign in to the AWS Console using the **spoke account**.
2.  Navigate to the **CloudFormation** console https://console.aws.amazon.com/cloudformation/
3.  Verify the region. 
4.  Choose **Create Stack**.
5.  Under Choose a template, select Specify an Amazon S3 template URL.
6.  Type the following URL  https://s3.amazonaws.com/scterraform-[YOUR-ACCOUNT-ID]/TerraformScripts/cloudformation-templates/terraform-spoke-principals.yaml
7.  Choose **Next**.
8.  For Stack name, type **TerraformLaunchRole**.
9.  For Fulfillment Account ID, type the hub account ID.
10. For FulfillmentRegion enter the region.
11. Update the remaining parameters (optional).
12. Choose **Next**.
13. Choose **Create stack**to create the Terraform instance.

The Status changes to CREATE_COMPLETE once the stack is created.

## Create the Terraform Lambda launch function in the regions you plan to use

**Note:** Make sure the TerraformLaunchRole CloudFormation stack has a status of CREATE_COMPLETE before proceeding. 
1.  Sign in to the AWS Console using the spoke account.
2.   Navigate to the CloudFormation console.
https://console.aws.amazon.com/cloudformation/
3.  Verify the region. 
4.  Choose **Create Stack.**
5.  Under Choose a template, select Specify an Amazon S3 template URL.
6.  Type the following URL:https://s3.amazonaws.com/scterraform-[YOUR-ACCOUNT-ID]/TerraformScripts/cloudformation-templates/terraform-launch-lambda.yaml
7.  Choose **Next**
8.  For **Stack name,** type **TerraformLaunchLambda.** 
9.  For **Fulfillment Account** ID** type the hub account ID.
10. For **FulfillmentRegion**  enter the region
11. Update the remaining parameters (optional).
12. Choose **Next.**
13. Choose **Next.**
14. Choose **Create** to create the Terraform instance.

The Status changes to CREATE_COMPLETE once the stack is created.

## Deploy Service Catalog portfolio and products.

For the test we will deploy the same SC portfolio and products in spoke account. To access the CFN templates from the spoke account, add required access policy on bucket. For this example, just make the files from the bucket __**terraformarchitecture-single-terraformconfigstore-xxxx**__ public. Go to S3 console of Hub account. Click on s3 bucket __**terraformarchitecture-single-terraformconfigstore-xxxx**__.  Select all the files and click on **Action** and the **Make public using ACL** so that spoke accounts can able to access them. 
:warning: In your production environment you will setup secure cross account S3 access policy and must not make the templates public. 

1. Navigate to the **CloudFormation console** https://console.aws.amazon.com/cloudformation/
2. Verify the **region**. 
3. Choose **Create Stack**.
9. Under Choose a template, select *Specify an Amazon S3 template URL*.
10. Copy the URL of json template **sc-sample-port-product-setup.json** from Hub account s3 bucket __**terraformarchitecture-single-terraformconfigstore-xxxx**__. Paste the URL you copied. 
12.  For Stack name, type **SCTFportfoliosetup**.
13.  For all the other parameters use the defaults. But for **TemplateConfigBucket** enter the S3 bucket full name which is similar to `terraformarchitecture-single-terraformconfigstore-xxxxx.s3.<s3-bucket-region>.amazonaws.com`
14. Choose **Next**.
15. Choose **Create Stack** to create the Service Catalog Sample Terraform Portfolio.

## Change SNS topic access policy

In order to publish the message on SNS topic `terraform-commands-topic` locate in Hub account from spoke account, you need to change the access policy of the SNS.
1. Go to Hub account and SNS console.
2. Search for SNS topic `terraform-commands-topic` and click on it.
3. Choose **Edit** and the **Access policy**. Add the ARN of newly created IAM role in Spoke account. i.e. `TerraformLaunchLambdaRole`
4. Click **Save changes**.

Now from the Spoke account's Service Catalog console you can launch the deployed products (by providing user/role access to deployed portfolio).

The above LambdaLaunch function, portfolio deployment steps needs to be repeated for each region.

---
## Using the AWS Service Catalog Terraform Reference Architecture with GitHub

This section is intended for users with a basic working knowledge of GitHub. For more information on GitHub, see https://developer.github.com/v3/guides/. 
To use the Terraform Reference Architecture with GitHub, follow these steps.  
1.  Create a MachineUser with access to the repositories you would like to reference: https://developer.github.com/v3/guides/managing-deploy-keys/#machine-users  

2.  Create a secret in your hub account that contains your MachineUser's private key. For example:  
    ```
    aws secretsmanager create-secret --secret-string "`cat id_rsa`" --name TerraformMachineUserIdentity
    ```  
    For more information, see https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html  

3.  Generate a hashed known_hosts file containing the public keys of your repository's host:  
    a.  Retrieve Github's public key using the following commands:  
    ```
    touch known_hosts
    chmod 600 known_hosts
    ssh-keyscan -t rsa github.com >> known_hosts
    ```
    b.  Verify the public key against Github's public key fingerprints using the following command:  
    ```cat known_hosts | ssh-keygen -lf -```  
    c.  Compare the output to the public key fingerprints Github has posted: https://help.github.com/articles/github-s-ssh-key-fingerprints/  
    d.  Generate a hashed known_hosts file using the public key:  
    ```ssh-keygen -H -f known_hosts```  

4.  Create a secret in your hub account that contains the hashed known_hosts file. For example:  
  ```aws secretsmanager create-secret --secret-string "`cat known_hosts`" --name TerraformKnownHosts```  

5.  Update the SshIdentitySecret and SshKnownHostsSecret parameters of your TerraformWrapperServer stack. For example:  
    ```
    aws cloudformation update-stack --template-url https://s3.amazonaws.com/scterraform-[YOUR-ACCOUNT-ID]/TerraformScripts/cloudformation-templates//terraform-fulfillment-server.yaml --parameters ParameterKey=SshKnownHostsSecret,ParameterValue=TerraformKnownHosts ParameterKey=SshIdentitySecret,ParameterValue=TerraformMachineUserIdentity --capabilities CAPABILITY_NAMED_IAM --stack-name TerraformWrapperServer
    ```


---
### Executing the Terraform file plan/dry run command
The Terraform plan command creates an execution plan or determines actions that are necessary to achieve the desired state, specified in the configuration files, without making any changes to real resources or to the state. The output of the plan is available in the output section of the AWS Service Catalog end user console. Include the DryRunId property and set the value to 1. The Terraform wrapper server will execute the plan as a preview only, instead of creating or managing AWS resources. 

    "Resources": {
            "MyTerraformStack": {
                "Type": "Custom::TerraformStack",
                "Properties": {
                    "ServiceToken": {
                        "Fn::Sub": "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:TerraformLaunchHandler"
                    },
                    "TerraformArtifactUrl": {
                        "Fn::Sub": https://s3.amazonaws.com/terraform-config-${AWS::AccountId}/sc-sample-lamp.tf 
                    },
                    "LaunchRoleArn": {
                        "Fn::Sub": "arn:aws:iam::${AWS::AccountId}:role/TerraformResourceCreationRole" 
                    },
                    "DryRunId": {“Ref”:“DryRunId”},  *

                    "TerraformVariables": {
                        "aws_region": {
                            "Fn::Sub": "${AWS::Region}"
                        },
                        "aws_ami": {
                            "Ref": "ImageID"

 - * Add this property for Plan or Dry run.

---

### FAQ
#### Can a Terraform wrapper template reference multiple Terraform configuration files?
Yes. If you have multiple Terraform configuration files, you can package them in a ZIP file, upload the ZIP file to Amazon S3, and then reference the ZIP file in your wrapper template. The Terraform Wrapper Server detects ZIP files and extracts their contents before attempting to execute any commands.
What are the limitations of this solution?
- Stack creation will fail for Terraform configurations that take longer than two hours to create.
- AWS is the only supported provider.
- CloudFormation plan APIs will not list the resources generated by a TerraformStack.

#### Can an IAM role be used to launch Terraform products instead of an IAM user?
Yes, as long as a role has permissions to provision AWS Service Catalog products and invoke the spoke account’s Lambda launch function, it can be used to launch Terraform products. For example, the CloudFormation template defines a role that can be used to launch Terraform products in a spoke account.

      Resources:
        TerraformUserLaunchRole:
          Type: AWS::IAM::Role
          Properties:
            RoleName: "TerraformLaunchLambdaRole"
            AssumeRolePolicyDocument:
              Version: 2012-10-17
              Statement:
                - Effect: Allow
                  Principal:
                    AWS:
                      - arn:aws:iam::111111111111:user/terraform-account-x-enduser
                  Action:
                    - sts:AssumeRole
            ManagedPolicyArns:
              - arn:aws:iam::aws:policy/AWSServiceCatalogEndUserFullAccess
            Policies:
              - PolicyName: TerraformLaunchLambdaPolicy
           PolicyDocument:
                  Statement:
                    - Effect: Allow
                      Action: lambda:InvokeFunction
                      Resource: !Sub arn:aws:lambda:*:${AWS::AccountId}:function:TerraformLaunchHandler
                    - Effect: Allow
                      Action: s3:GetObject
                      Resource: '*'


### Security Best Practices

**Update your Terraform binary version when you identify vulnerabilities**

If you become aware of a security vulnerability affecting the Terraform binary version installed on your Terraform Wrapper Server, switch to a version that is not affected by the vulnerability. To do this, update your Terraform Wrapper Server stack by setting the stack's TerraformVersion parameter to the version you want to switch to. Keep in mind that you may need to update your Terraform configurations if a backwards incompatible change has been introduced in the Terraform binary version to which you are switching.

**Restrict access to the Launch Lambda in the spoke account**

Because the Lambda launch function has access to the hub account’s SNS topic, you should restrict access to the Lambda launch function in the spoke account. Do not grant non-administrator users in the spoke account permissions to lambda:UpdateFunctionCode , lambda:InvokeFunction or lambda:UpdateFunctionConfiguration unless you exclude the Lambda launch function using a resource element. 

**Do not remove the ExternalId from the spoke account’s ResourceCreationRole**

The spoke account’s IAM Resource Creation Role uses an ExternalId to limit access to the spoke account. You should retain this ExternalId to help prevent inadvertent modification to resources in the spoke account by users that have access to the hub account’s Terraform Server Role.

**Limit access to the hub account**

The hub account’s Terraform Wrapper Server will have access to Resource Creation Roles in multiple spoke accounts. To limit the exposure of these roles, you should restrict access to the hub account as much as possible and use the hub account for Terraform functionality only. 

## Questions and Comments  
aws-sa-servicecatalog@amazon.com


### Further Reading
- https://www.terraform.io/intro/index.html

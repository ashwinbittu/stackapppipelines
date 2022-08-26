<div id="top"></div>


<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#about-the-project">About The Project</a></li>
    <li><a href="#prerequisites">Prerequisites</a></li>
    <li><a href="#installation">Installation</a></li>
    <li><a href="#How-To-Use-The-Script">How To Use The Script</a></li> 
    <li><a href="#Medium-Links">Medium Links</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

In this project we deploy the `App layer` of <a href="https://github.com/ashwinbittu/terraform-aws-ec2-contino">Conitno Sample Application</a> on an existing VPC in ap-southeast-2 region. We will be using Terraform Cloud account for storing the ENV state and its provisioning purposes. All the Terraform calls in the triggering script here are based on <a href="https://github.com/hashicorp/terraform-guides/tree/master/operations/automation-script">`Terraform REST APIs developed by Hashicorp`</a> and NOT CLI. Please have a look these automation scripts. 

As part of this the following AWS resources are created:

* The script uses existing Terraform Module <a href="https://github.com/ashwinbittu/terraform-aws-ec2-contino">`ec2`</a>  to provision 3 `t3.micro` EC2 instances in the following Availability Zones and Subnets.

    |    Subnet    | Availability Zone |
    |--------------|-------------------|
    | subnet-az-2a |  ap-southeast-2a  |
    | subnet-az-2b |  ap-southeast-2b  |
    | subnet-az-2c |  ap-southeast-2c  |

* The script also uses existing Terraform Module <a href="https://github.com/ashwinbittu/terraform-aws-key-pair-contino">`key-pair`</a> to provision a Key pair as well.
* We will be deploying a publicly available AMI which just has a basic static home page with an image running on Apache Web Server(on port 80) on all these 3 instances & once we complete this deployment then we also do a very basic check of Web App's home page availability.

Following is the high level diagram of what we are trying to achieve here:

![highlevel](https://miro.medium.com/max/1400/1*Crsuk5dqfReuoX6nwXVatQ.png "highlevel")  

<br>

Following are the input parameters while running the script:

- Instance Type: `t3.micro`
- Tags: Add a `Name` tag that is unique for each instance.

Following are the output values after running the script:

1. List of three 3 EC2 Instance IDs and its Names
2. Map value of the Key Name, its Private and Public Keys.

<p align="right">(<a href="#top">back to top</a>)</p>



### Prerequisites

* You already have an AWS free tier account with Admin access
* You already have a GitHub account. 
* You already have a Terraform Cloud free tier account. Please follow this link to do the same: https://app.terraform.io/public/signup/account 
* You already created an API Token from the Users section Terraform Cloud free tier account. Go to https://app.terraform.io/app/settings/tokens → Create API Token. Keep it safe, we will be using it in the automated create and destroy infra shell script.
* You already have created an Organization in Terraform Cloud free tier account. Go to https://app.terraform.io/app/organizations → Click on Create New Organization
* You already have created the following `Variable Set` & the 4 variables in your Terraform Cloud Organization. Please follow this link to find out how: 
    * Variable Set Name: `contino-global-vars`
    * Variable Details:
        |        Name       |                            Value                          | HCL Tick |   Type    |
        |-------------------|-----------------------------------------------------------|----------|-----------|
        | availability_zone |  ["ap-southeast-2a","ap-southeast-2b","ap-southeast-2c"]  |   Yes    | Terraform |
        |    subnet_name    |        ["subnet-az-2a","subnet-az-2b","subnet-az-2c"]     |   Yes    | Terraform |
        |      tf_host      |                   app.terraform.io                        |    No    | Terraform |
        |      tf_org       |                   Your-Terraform-ORG-Name                 |    No    | Terraform |

  ![vars](https://miro.medium.com/max/1400/1*YH_M7gyE724LVdWHCxs6Hw.png "vars")      

* You already have imported or downloaded & checked-In the following repos into your GitHub account:
    * <a href="https://github.com/ashwinbittu/terraform-aws-ec2-contino">`ec2`</a>
    * <a href="https://github.com/ashwinbittu/terraform-aws-key-pair-contino">`key-pair`</a>
    * <a href="https://github.com/ashwinbittu/continoapp">`Conitno Sample Application`</a>

    ***An Important Note: PLEASE DOWNLOAD & CHECK-IN THE ABOVE THREE REPOS[EC2, KEYPAIR & CONTINO APP] IN YOUR GITHUB ACCOUNT, ADDING YOUR ACCOUNT AS COLLABORATER TO THESE REPOS WILL HAVE ISSUES WHILE PUBLISHING THESE MODULES INSIDE TERRAFORM CLOUD, TO AVIOD THIS, YOU MUST DOWNLOAD & CHECK-IN THESE REPOS IN YOUR GITHUB ACCOUNT***
* You have made the following two changes in https://github.com/Your-GitHub-ID/continoapp/blob/main/application/main.tf and Commit the file:
    * source  = "app.terraform.io/Continopoc/key-pair-contino/aws": Replace `Continopoc` with your Terraform Organization Name.
    * source  = "app.terraform.io/Continopoc/ec2-contino/aws"	: Replace `Continopoc` with your Terraform Organization Name.
* You already have Tagged these repos to some initial version say v1.0.0. Tagging is important for publishing these modules in Terraform Cloud. Click on Create a new release then put a tag name for ex: v1.0.0. then click on Create new tag, after than click on Publish release.
    ![Tagging](https://miro.medium.com/max/875/1*qoL48vAlq2J4dwtyfeaZWQ.png  "Tagging")
* You already have/Created Personalized Access Token to access your GitHub repos. For doing this, go to https://github.com/settings/tokens → Generate New Token. Copy the Token string and keep it safe, we will be using it later.
    ![patoken](https://miro.medium.com/max/875/1*3x4HcG4ZT68BNKMZIBtBQg.png  "patoken")
* You already have integrated/Connected your Terraform Organization with the above GitHub(Repo). 
    * Go To https://app.terraform.io/app/<YOUR-TF-ORG>/settings/version-control → Add a VCS Provider
        
        ![output](https://miro.medium.com/max/875/0*_NsRRQBhsigBM3fE.png  "output")
    * Choose GitHub.com then, login to your GitHub account then go to https://github.com/settings/applications/new. Register for a new OAuth using the details provided in the left side.
        
        ![output](https://miro.medium.com/max/875/0*UfMsQ3RsM3PiX5t0.png  "output")
    * After registering the application, create a secret and paste all the entries in the empty fields in the same page at the Terraform Cloud side.
        
        ![output](https://miro.medium.com/max/875/0*ogsxKOeN1RP41v3w.png  "output")
    You can skip the SSH key mapping and finish it. Once you have successfully created this link then all your Terraform Modules will be visible for registering and publishing in the Terraform Cloud.
* You already have published both the modules terraform-aws-ec2-contino & terraform-aws-key-pair-contino in your Organization's registry. 
    * Register VCS & Publish Terraform Modules: Go To https://app.terraform.io/app/<YOUR-TF-ORG>/registry/modules/add → Choose the recently mapped GitHub VCS provider
        
        ![output](https://miro.medium.com/max/875/0*gaZbXPtMRz5FzMlX.png  "output")
    * You already have published both the modules terraform-aws-ec2-contino & terraform-aws-key-pair-contino in your Organization’s registry.
        
        ![output](https://miro.medium.com/max/875/1*wzmQKo3Ykjc9bHff96yf1Q.png  "output")
* You already have an existing AWS_ACCESS_KEY_ID & AWS_SECRET_ACCESS_KEY pair which can be used to access the AWS ENV through any trigger script. 
* You have Putty Installed on your Desktop
* You have WinSCP Installed on your Desktop


### Installation

1. Get a `t2.micro` Amazon Linux EC2 Instance(This is for Running the script). Logon to the instance using Putty.
2. Install jq : 
   ```sh
   sudo yum install jq -y
      ```
3. Install git : 
   ```sh
   sudo yum install git -y
      ```
4. Clone the repo
   ```sh
   git clone https://github.com/ashwinbittu/managecontinoinfra.git
      ```

   OR download the zip file and transfer it to the EC2 Instance through WinSCP  
4. Give appropriate permission(chmod command) to the file manageinfra.sh for executing it.
5. Open the file manageinfra.sh in vi editor & update the following entries with the values already mentioned in the <a href="#Prerequisites">Prerequisites</a> section.
    * TFE_TOKEN=`TFE_TOKEN`
    * TFE_ORG=`TFE-ORG`
    * REPO_API_TOKEN=`GITHUB_Personal Access Token`
    * REPO_FID=`GITHUB_USER_ID`
    * AWS_ACCESS_KEY_ID=`AWS_ACCESS_KEY_ID`
    * AWS_SECRET_ACCESS_KEY=`AWS_SECRET_ACCESS_KEY`

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- USAGE EXAMPLES -->
### How To Use The Script

All the json files that are part of this repo for ex: apply.json, run.template.json etc. These are input files to the Terraform REST API calls.

There are currently two actions that we can perform with the script manageinfra.sh:

1. Create Infra:  
   ```sh
   ./manageInfra.sh create <ec2-instnace-type> <server-name-of-choice> , For example: ./manageInfra.sh create t3.micro continoserver   
      ```


The above command does the following actions, all these are Terraform REST API call and not CLI:

* First It creates a Terraform Workspace with the name `continoapp-dev-ap-southeast-2-application`, here:
    * `continoapp`: Is the name of the app
    * `dev`: Is the App ENV, for UAT and PROD, the values would have been, `continoapp-uat-ap-southeast-2-application` or `continoapp-prod-ap-southeast-2-application`
    * `ap-southeast-2`: Is the target AWS region
    * `application`: Is the App Tier. If we were creating Network or Storage resources then the workspace names would have been `continoapp-dev-ap-southeast-2-network` & `continoapp-dev-ap-southeast-2-storage`. This segregation is done to individually control each layer of app infra. 

    **If you want to know how each App layer is seggerated according to recommendations from HashiCrop, then please visit this section of the link:** https://medium.com/@ashwin.bittu/terraform-testing-e80f6316875a#eb58

    All the above values are already part of manageInfra.sh script.

* Creates Workspace variables available in the file, https://github.com/Your-GitHub-ID/continoapp/blob/main/application/dev/ap-southeast-2.csv, here we have variables for each region and for each environment, here for example its dev.
* Creates the following Workspace variables:
    * `AWS_ACCESS_KEY_ID` & `AWS_SECRET_ACCESS_KEY`
    * `AWS_REGION`
    * `aws_region`
* Updates the Workspace variables `instance_type` & `name_tag` based on the arguments provided.
* Uploads the Terraform Configuration to Terraform Cloud Account. Here configuration means the https://github.com/Your-GitHub-ID/continoapp code.
* Executes a `Run` action on the Terraform Workspace, Here `Run` is the combination of first running `terraform plan` and then running `terraform apply`.

![consoleoutput](https://miro.medium.com/max/1400/1*YOggr9szH6r148k0H3hG7Q.png  "consoleoutput")


![statefile](https://miro.medium.com/max/1400/1*GYglyRDwRaPe9U2E2cPCbQ.png  "statefile")


* After the `Run` execution, the infra is created and the following output is given:
    ![output](https://miro.medium.com/max/1400/1*ifNml3Ve58ItZERuSCBlXQ.png  "output")
    
* After the successful `Run` execution, the script also checks if the App URL is up and running, hence printing out the following last log(here the IP will differ). To see if the sample app is running, one our 3 Instance IPs are checked at port 80:
   ```sh
   Contino App is Up and Running, please check the URL: http://13.211.153.232  

  Home Page URL image:

![homepage](https://miro.medium.com/max/1400/1*TxBvL2x1hkdEtxa5mvtZAw.png  "homepage")

2. Destroy Infra: 
   ```sh
   ./manageInfra.sh destroy  
      ```
This commands just destroys the whole infra along with the workspace

![destroy](https://miro.medium.com/max/1400/1*VpxyplT5N5dAk965fm3y5g.png  "destroy")

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- MEDIUM LINKS -->
### Medium Links

Please follow these links for more info:

<p align="right">(<a href="#top">back to top</a>)</p>


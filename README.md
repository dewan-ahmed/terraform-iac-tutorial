# Infrastructure as Code Tutorial: Use Terraform to build, configure, and destroy data infrastructure.

Infrastructure as Code (IaC) is a software engineering approach that enables the management and provisioning of infrastructure using code. Terraform is an open-source infrastructure as code (IaC) tool that allows you to create, manage, and update your infrastructure in a declarative way. It uses a configuration language called HashiCorp Configuration Language (HCL) or JSON to define your infrastructure resources, and then applies those configurations to your cloud provider to create and manage your infrastructure.

There are thousands of Terraform providers in the community to help you build and manage many different types of cloud resources. Since our focus is to build data infrastructure, we'll be using [Aiven](https://aiven.io/) as the platform. 

In this tutorial, we will use Terraform and [Aiven Terraform Provider](https://registry.terraform.io/providers/aiven/aiven/latest/docs) to create and manage data infrastructure on Aiven.

## The challenge

Today is the first day at new job for Mr. Byte Boss. He is joining as the head of engineering for a tech company that has engineering teams in USA, UK, and Singapore. He is tasked to solve the cloud deployment mess in the org. All the engineering teams are maintaining their own shell scripts to deploy cloud resources. Development environments don't have parity and testing is inaccurate due to the environment mismatch between development and production. Furthermore, new engineers joining the team have a hard time to understand various complex shell scripts. Some engineers create cloud resources directly on the console which makes the challenge even harder.

Your goal is to prepare a proof-of-concept with Terraform to help Mr. Byte Boss untangle the deployment mess in his org.  

## Prerequisites

For this tutorial, you need to [download](https://www.terraform.io/downloads.html) and [install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli#install-terraform).

You'll also create a [free Aiven account](https://console.aiven.io/signup) and [an Aiven authentication token](https://docs.aiven.io/docs/platform/howto/create_authentication_token). 

## Challenge 1: Declarative infrastructure

Your first challenge is to convince the teams to get rid of all the complex shell scripts and use an IaC tool like Terraform for deployment. Let's start small and create a Redis service on an AWS USA region using Terraform. 

Your Terraform files declare the structure of your infrastructure as well as required dependencies and configuration. While you can stuff these together in one file, it's ideal to keep those as separate files. 

### Define the required Terraform files

Start with an empty folder, and follow these steps to define and then create an Aiven for Redis service.

1. First we declare a dependency on the **Aiven Terraform Provider**. Within the **required_providers** block, you mention the source of the provider and specify a certain version (check which are the current versions and update accordingly).
Following [Aiven Terraform Provider documentation](https://registry.terraform.io/providers/aiven/aiven/latest/docs), **api_token** is the only parameter for the provider configuration.

Add the following to a new **provider.tf** file:

```
   terraform {
     required_providers {
       aiven = {
         source  = "aiven/aiven"
         version = "~> 4.1.0"
       }
     }
   }
   
   provider "aiven" {
     api_token = var.aiven_api_token
   }
```

You can also set the environment variable **TF_VAR_aiven_api_token** for the **api_token** property and **TF_VAR_project_name** for the **project_name** property. With this, you don't need to pass the **-var-file** flag when executing Terraform commands.
 
2. To avoid including sensitive information in source control, the variables are defined in the **variables.tf** file. You can then use a ***.tfvars** file with the actual values so that Terraform receives the values during runtime, and exclude it.

The **variables.tf** file defines both the API token, and the project name to use:

```
   variable "aiven_api_token" {
     description = "Aiven console API token"
     type        = string
   }
   
   variable "project_name" {
     description = "Aiven console project name"
     type        = string
   }
```   
   
The **var-values.tfvars** file holds the actual values and is passed to Terraform using the **-var-file=** flag.

**var-values.tfvars** file:

```
   aiven_api_token = "<YOUR-AIVEN-AUTHENTICATION-TOKEN-GOES-HERE>"
   project_name    = "<YOUR-AIVEN-CONSOLE-PROJECT-NAME-GOES-HERE>"
```

Edit the file and replace the **<..>** sections with the API token you created earlier, and the name of the Aiven project that resources should be created in.

3.  The following Terraform script creates a single-node Redis service on the Aiven platform.

The contents of the **redis.tf** file should look like this:

```
    # A single-node Redis service
    
    resource "aiven_redis" "single-node-aiven-redis" {
      project                 = var.project_name
      cloud_name              = "aws-us-east-1"
      plan                    = "hobbyist"
      service_name            = "aws-us-redis"
    }
```   
    
### Apply the Terraform configuration

The **init** command performs several different initialization steps in order to prepare the current working directory for use with Terraform. In our case, this command automatically finds, downloads, and installs the necessary Aiven Terraform provider plugins.

```
   terraform init 
```

The **plan** command creates an execution plan and shows you the resources that will be created (or modified) for you. This command does not actually create any resource; this is more like a preview.

```
   terraform plan -var-file=var-values.tfvars
```

If you're satisfied with the output of **terraform plan**, go ahead and run the **terraform apply** command which actually does the task or creating (or modifying) your infrastructure resources. 

```
   terraform apply -var-file=var-values.tfvars
```

You'll need to type in **yes** to confirm. 

The output will show you if everything worked well. You can now visit the [Aiven web console](https://console.aiven.io) and observe the service creation. A flashing blue sign beside your service indicates that your service is being provisioned. A solid green sign indicates that the service is up and running.

### Clean up

Be sure to delete the resources once you're done to avoid consuming unwanted bills. 

```
    terraform destroy -var-file=var-values.tfvars
```

You'll need to type in **yes** to confirm.

### What did we get out of this POC?

With a shell script, you typically don't have idempotence. That means, if you run the script 5 times, either you'll have 5 services created if you used a random UUID for the name, or you'll get an error that a service name with the specified name already exists. 

With Terraform, regardless of how many times you run **terraform apply**, your services will be created only once. However, if the service state deviates from the expected state, Terraform will ensure that it applies the desired state back. 
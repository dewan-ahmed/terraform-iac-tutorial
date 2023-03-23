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

## Challenge 2: Repeatable infrastructure

Mr. Boss' next challenge is to bring standardization within his engineering teams. During the testing for the last product release, the US team used PostgreSQL version 12, the UK team used version 13, and the Singapore team used version 14. Although the blesses shell script is stored in version control, most teams work of their own copy of that script with plenty of customization for machine types and configurations.

Your goal for this challenge is to help Mr. Boss bring order to the chaos among three engineering teams and propose standardization using Terraform and Aiven. The tech stack for the company is

- Aiven for PostgreSQL service for relational data
- Aiven for Redis for non-relational or NoSQL data
- Aiven for ClickHouse for analytical data

Besides creating the services, you'll also need to create a database and a database admin user for the Aiven for PostgreSQL service, a service inegration to the Aiven for ClickHouse service so that the database from the PostgreSQL service is copied over to ClickHouse for analytical processing. Service integration is an Aiven concept that allows you to connect two Aiven services to exchange data without you having to glue them together with your own code. Finally, you'll need to create an admin user for the Redis service.

![Development environment blueprint](/images/dev-env-blueprint.png)

Let's create the blueprint for these services and resources.

### Define the required Terraform files

At first, create an empty directory and add the common files for this project.

1. Create a file called **provider.tf** which declares the required Aiven Terraform Provider and specific version number.

```
terraform {
  required_providers {
    aiven = {
      source  = "aiven/aiven"
      version = "~> 3.9.0"
    }
  }
}

provider "aiven" {
  api_token = var.aiven_api_token
}
```

2. Create a file called **variables.tf** to declare some input variables. Input variables in Terraform are like function arguments of traditional programming languages.

```
variable "aiven_api_token" {
  description = "Aiven console API token"
  type        = string
}

variable "project_name" {
  description = "Aiven console project name"
  type        = string
}

variable "db_username" {
  description = "Database administrator username"
  type        = string
  sensitive   = true
}

variable "db_password" {
  description = "Database administrator password"
  type        = string
  sensitive   = true
}
```

3. The actual values of the **variables.tf** file will come from a ***.tfvars** file. Create a file called **dev.tfvars** and add the following:

```
aiven_api_token = YOUR_API_TOKEN_GOES_HERE
project_name    = YOUR_AIVEN_PROJECT_NAME_GOES_HERE
db_username     = "admin"
db_password     = "insecurepassword"
```

Because this is a demo, you'll be using the same database username/password for PostgreSQL and Redis. Ideally, you would access your databases using some sort of IAM or secret management tools and not using the root database credentials.

4. The actual Terraform resources are defined in a **services.tf** file. Create this file within the same folder and add the following:

```
// This creates the PostgreSQL service
resource "aiven_pg" "postgres_service" {
  project                 = var.project_name
  service_name            = "postgres-aws-us"
  cloud_name              = "aws-us-east-1"
  plan                    = "startup-4"
}

// This creates the relational PostgreSQL database
resource "aiven_pg_database" "relational_database" {
  project       = var.project_name
  service_name  = aiven_pg.postgres_service.service_name
  database_name = "relational_database"
}

// This creates the PostgreSQL admin user
resource "aiven_pg_user" "postgres_admin_user" {
  service_name = aiven_pg.postgres_service.service_name
  project      = var.project_name
  username     = var.db_username
  password     = var.db_password
}

// This creates the ClickHouse service
resource "aiven_clickhouse" "clickhouse_service" {
  project                 = var.project_name
  service_name            = "clickhouse-aws-us"
  cloud_name              = "aws-us-east-1"
  plan                    = "startup-8" 
}


// ClickHouse service integration for the PostgreSQL service as a source
resource "aiven_service_integration" "clickhouse_postgres_source" {
  project                  = var.project_name
  integration_type         = "clickhouse_postgresql"
  source_service_name      = aiven_pg.postgres_service.service_name
  destination_service_name = aiven_clickhouse.clickhouse_service.service_name
  clickhouse_postgresql_user_config {
    databases {
      database = aiven_pg_database.relational_database.database_name
      schema   = "public"
    }
  }
}

// This creates the Redis service
resource "aiven_redis" "redis_service" {
  project                 = var.project_name
  cloud_name              = "aws-us-east-1"
  plan                    = "startup-4"
  service_name            = "redis-aws-us"
}

// This creates the Redis admin user
resource "aiven_redis_user" "redis_admin_user" {
  service_name = aiven_redis.redis_service.service_name
  project      = var.project_name
  username     = var.db_username
  password     = var.db_password
}
```

5. If input variables are like function argument, output values in Terraform are like function return values. Create a file called **outputs.tf** and add the following:

```
output "postgres_connect_string" {
  description = "Postgres database connection string"
  value       = aiven_pg.postgres_service.service_uri
  sensitive   = true
}

output "redis_connect_string" {
  description = "Redis database connection string"
  value       = aiven_redis.redis_service.service_uri
  sensitive   = true
}
```

Note that since these output values contain database credentials, these values are marked as **sensitive**. Terraform will hide values marked as sensitive in the console output. You can find the values under the Terraform state file. 

    
### Apply the Terraform configuration

The **init** command performs several different initialization steps in order to prepare the current working directory for use with Terraform. In our case, this command automatically finds, downloads, and installs the necessary Aiven Terraform provider plugins.

```
   terraform init 
```

The **plan** command creates an execution plan and shows you the resources that will be created (or modified) for you. This command does not actually create any resource; this is more like a preview.

```
   terraform plan -var-file=variables.tfvars
```

If you're satisfied with the output of **terraform plan**, go ahead and run the **terraform apply** command which actually does the task or creating (or modifying) your infrastructure resources. 

```
   terraform apply -var-file=variables.tfvars
```

You'll need to type in **yes** to confirm. 

The output will show you if everything worked well. You can now visit the [Aiven web console](https://console.aiven.io) and observe the services being created. A flashing blue sign on the Aiven console beside your service will indicate that your service is being provisioned. A solid green sign will indicate that the service is up and running. Alternatively, you can find the service URIs for the PostgreSQL and Redis services from the Terraform state file (as a result of the **outputs.tf** file) and use those programmatically to connect to these services. 

### Clean up

Since this was a test environment, be sure to delete the resources once you're done to avoid consuming unwanted bills. 

```
    terraform destroy -var-file=variables.tfvars
```

You'll need to type in **yes** to confirm.
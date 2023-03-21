# Infrastructure as Code Tutorial: Use Terraform to build, configure, and destroy data infrastructure.

Infrastructure as Code (IaC) is a software engineering approach that enables the management and provisioning of infrastructure using code. Terraform is an open-source infrastructure as code (IaC) tool that allows you to create, manage, and update your infrastructure in a declarative way. It uses a configuration language called HashiCorp Configuration Language (HCL) or JSON to define your infrastructure resources, and then applies those configurations to your cloud provider to create and manage your infrastructure.

There are thousands of Terraform providers in the community to help you build and manage many different types of cloud resources. Since our focus is to build data infrastructure, we'll be using [Aiven](https://aiven.io/) as the platform. 

In this tutorial, we will use Terraform and [Aiven Terraform Provider](https://registry.terraform.io/providers/aiven/aiven/latest/docs) to create and manage data infrastructure on Aiven.

## The challenge

Today is the first day at new job for Mr. Byte Boss. He is joining as an IT director for a tech company that has engineering teams in USA, UK, and Singapore. He is tasked to solve the cloud deployment mess in the org. All the engineering teams are maintaining their own shell scripts to deploy cloud resources. Development environments don't have parity and testing is inaccurate due to the environment mismatch between development and production. Furthermore, new engineers joining the team have a hard time to understand various complex shell scripts. Some engineers create cloud resources directly on the console which makes the challenge even harder.

Your goal is to prepare a proof-of-concept with Terraform to help Mr. Byte Boss untangle the deployment mess in his org.  

## Prerequisites

For this tutorial, you need to [download](https://www.terraform.io/downloads.html) and [install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli#install-terraform).

You'll also create a [free Aiven account](https://console.aiven.io/signup) and [an Aiven authentication token](https://docs.aiven.io/docs/platform/howto/create_authentication_token). 
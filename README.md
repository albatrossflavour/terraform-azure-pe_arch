# azure-pe_arch

IaC definitions for three of the supported Puppet Enterprise architectures for Azure

## Table of Contents

1. [Description](#description)
2. [Setup - The basics of getting started with azure-pe_arch](#setup)
    * [What azure-pe_arch affects](#what-azure-pe_arch-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with terraform-azure-pe_arch](#beginning-with-terraform-azure-pe_arch)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Limitations - OS compatibility, etc.](#limitations)

## Description

This Terraform module implements as code the infrastructure required to deploy three permutations of the [supported](https://puppet.com/docs/pe/latest/choosing_an_architecture.html) Puppet Enterprise architectures: Standard, Large, and Extra Large, addtionally all architectures can have additional infrastructure provisioned to support a failover replica on Azure. This module is developed to function independently but it is often used in support of [puppetlabs/peadm](https://github.com/puppetlabs/puppetlabs-peadm), brought together by [puppetlabs/autope](https://github.com/puppetlabs/puppetlabs-autope) to facilitate the end-to-end deployment of fully functional stacks of Puppet Enterprise. It sets up native azure networking and load balancers specifically for containing and managing access to the deployment but avoids a dependence on cloud provided SQL services since Puppet Enterprise has its own facilities for managing and automating PostgreSQL.

### Expectations and support

This Terraform module is intended to be used only by Puppet Enterprise customers actively working with and being guided by Puppet Customer Success teams—specifically, the Professional Services and Solutions Architecture teams. Independent use is not recommended for production environments without a comprehensive understanding of how Terraform works, comfort in the modification and maintenance of Terraform code, and the infrastructure requirements of a full Puppet Enterprise deployment.

This Terraform module is a services-led solution, and is **NOT** supported through Puppet Enterprise's standard or premium support.puppet.com service.

As a services-led solution, Puppet Enterprise customers who are advised to start using this module should get support for it through the following general process.

1. Be introduced to the module through a services engagement or by their Technical Account Manager (TAM).
2. During Professional Services (PS) engagements, the Puppet PS team will aid and instruct in use of the module.
3. Outside of PS engagements, use TAM services to request assistance with problems encountered when using the module, and to inform Puppet Customer Success (CS) teams of planned major maintenance or upgrades for which advisory services are needed.
4. In the absence of a TAM, your Puppet account management team (Account Executive and Solutions Engineer) may be a fallback communication option for requesting assistance, or for informing CS teams of planned major maintenance for which advisory services are needed.

This module is under active development and yet to release an initial version. There is no guarantee yet on a stable interface from commit to commit and those commits may include breaking chnages.

## Setup

### What azure-pe_arch affects

Types of things you'll be paying your cloud provider for

* Instances of various sizes
* Load balancers
* Networks

### Setup Requirements

* [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
* [AZ Login](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs#authenticating-to-azure)
* [Git Installed](https://git-scm.com/downloads)
* [OpenTofu (>= 1.5) Installed](https://opentofu.org/docs/intro/install/) or [Terraform (>= 1.5)](https://developer.hashicorp.com/terraform/install)
* An Azure subscription ID available to the provider (see below)

The module pins these provider versions:

* `hashicorp/azurerm` 5.7.0
* `hashicorp/random` 3.9.1
* `chriskuchin/hiera5` 0.5.4

### Selecting the Azure subscription

Since azurerm 4.0 the provider must be given a subscription ID. The module accepts an optional `subscription_id` variable. When it is left unset the provider falls back to the `ARM_SUBSCRIPTION_ID` environment variable, and then to the default subscription of the logged in Azure CLI. When driving the module from pecdm the simplest option is to export `ARM_SUBSCRIPTION_ID` before running the plan, because the environment variable is also picked up by the destroy plan, which writes its own minimal tfvars file.

```bash
export ARM_SUBSCRIPTION_ID=$(az account show --query id -o tsv)
```

azurerm 5.0 no longer registers Azure resource providers by default. The module registers `Microsoft.Compute` and `Microsoft.Network` itself, which only needs permission to register resource providers if they are not already registered in the subscription.

### Beginning with terraform-azure-pe_arch

1. Clone this repository
    * `git clone https://github.com/puppetlabs/terraform-azure-pe_arch.git && cd terraform-azure-pe_arch`
2. Install module dependencies: `terraform init`
3. Initiate plan for the default standard architecture
    * `terraform apply -auto-approve -var "project=example.com" -var "user=john.doe" -var "firewall_allow=[ \"0.0.0.0/0\" ]"`
4. Moments later you'll be presented with a single VM where to install Puppet Enterprise

## Usage

### Example: deploy large architecture with replica and a more restrictive network

This will give you the absolute minimum needed for installing Puppet Enterprise, a single VM plus a specific network for it to reside within and limited to a specific network that have access to the new infrastructure (note: internal network will always be injected into the list)

`terraform apply -auto-approve -var "project=example.com" -var "user=john.doe" -var "firewall_allow=[ \"192.69.65.0/24\" ]" -var "architecture=large" -var "replica=true"`

### Example: destroy stack

The number of options required are reduced when destroying a stack

`terraform destroy -auto-approve -var "project=example.com" -var "user=john.doe"`

## Usage notes

1. For making ssh access work with Terraform's Azurerm provider, you will need to add your private key corresponding to the public key in the `ssh_key` parameter to the ssh agent like so:

```bash
> eval `ssh-agent`
> ssh-add <private_key_path>
```

## Limitations

The default Linux image is AlmaLinux 9 (`almalinux:almalinux-x86_64:9-gen2:latest`), which needs no Marketplace plan, so `image_plan` defaults to an empty string. Any other image can be passed through `instance_image`, with `image_plan` set when the image carries a Marketplace plan. VM disk sizes are not configurable.

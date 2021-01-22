---
description: Learning IAC
---

# My Notes: learn Terraform Modules

{% embed url="https://learn.hashicorp.com/tutorials/terraform/module?in=terraform/modules" caption="" %}

Some of the content is copied as my notes from the Terraform Modules Overview:

"A Terraform module is a set of Terraform configuration files in a single directory. Even a simple configuration consisting of a single directory with one or more `.tf` files is a module. When you run Terraform commands directly from such a directory, it is considered the **root module**.

1. Start writing your configuration with modules in mind. 
2. Use local modules to organize and encapsulate your code. 
3. Use the public Terraform Registry to find useful modules. 
4. Publish and share modules with your team. 

_...we recommend that every Terraform configuration be created with the assumption that it may be used as a module, because doing so will help you design your configurations to be flexible, reusable, and composable._..."

Modules are downloaded when running `terraform init`

![](https://johnthompson365.com/wp-content/uploads/2020/12/image-14.png)

Stored locally:

![The directory structure from my ](https://johnthompson365.com/wp-content/uploads/2020/12/image-16.png)

## Terraform Files and how they relate to modules <a id="block-3e90f5e5-58b0-47c5-903c-60f79d1107fb"></a>

![](https://johnthompson365.com/wp-content/uploads/2020/12/image-18.png)

Each of these files serves a purpose:

* [`LICENSE`](https://learn.hashicorp.com/tutorials/terraform/module-create?in=terraform/modules#license) will contain the license under which your module will be distributed. When you share your module, the LICENSE file will let people using it know the terms under which it has been made available. Terraform itself does not use this file.
* [`README.md`](https://learn.hashicorp.com/tutorials/terraform/module-create?in=terraform/modules#readme-md) will contain documentation describing how to use your module, in markdown format. Terraform does not use this file, but services like the Terraform Registry and GitHub will display the contents of this file to people who visit your module's Terraform Registry or GitHub page.
* [`main.tf`](https://learn.hashicorp.com/tutorials/terraform/module-create?in=terraform/modules#main-tf) will contain the main set of configuration for your module. You can also create other configuration files and organize them however makes sense for your project.
* [`variables.tf`](https://learn.hashicorp.com/tutorials/terraform/module-create?in=terraform/modules#variables-tf) will contain the variable definitions for your module. When your module is used by others, the variables will be configured as arguments in the `module` block. Since all Terraform values must be defined, any variables that are not given a default value will become required arguments. Variables with default values can also be provided as module arguments, overriding the default value.
* [`outputs.tf`](https://learn.hashicorp.com/tutorials/terraform/module-create?in=terraform/modules#outputs-tf) will contain the ouput definitions for your module. Module outputs are made available to the configuration using the module, so they are often used to pass information about the parts of your infrastructure defined by the module to other parts of your configuration.

There are also some other files to be aware of, and ensure that you don't distribute them as part of your module:

* [`terraform.tfstate`](https://learn.hashicorp.com/tutorials/terraform/module-create?in=terraform/modules#terraform-tfstate) and `terraform.tfstate.backup`: These files contain your Terraform state, and are how Terraform keeps track of the relationship between your configuration and the infrastructure provisioned by it.
* [`.terraform`](https://learn.hashicorp.com/tutorials/terraform/module-create?in=terraform/modules#terraform): This directory contains the modules and plugins used to provision your infrastructure. These files are specific to a specific instance of Terraform when provisioning infrastructure, not the configuration of the infrastructure defined in `.tf` files.
* [`*.tfvars`](https://learn.hashicorp.com/tutorials/terraform/module-create?in=terraform/modules#tfvars): Since module input variables are set via arguments to the `module` block in your configuration, you don't need to distribute any `*.tfvars` files with your module, unless you are also using it as a standalone Terraform configuration.

**WHAT DOES THIS MEAN?**  
Terraform loads all configuration files within a directory and appends them together, which means that any resources or providers with the same name in the same directory will cause a validation error. If you were to run a terraform command now, your `resource "random_pet"` and `provider "aws"` block could cause errors.

In my example it means it appends any **.tf** files when applying. So if you create a **windows.tf** and **linux.tf** there can't be any resources or providers with the same name in them or you get complaints!

## Separate States

The destroy you just ran got rid of resources from both development and production. While you could use the `terraform taint` command to specify which resources you need to recreate individually, that approach is error-prone and requires more work. To avoid having to individually taint resources, you need to separate your development and production state.

State separation signals more mature usage of Terraform; with additional maturity comes additional complexity. There are two primary methods to separate state between environments: directories and _workspaces_.

## Directory:

**Rule of Thumb:** To separate environments with potential configuration differences, use a directory structure.

1. You have to navigate between directories
2. Complete separation so can run different configurations easily

![](https://johnthompson365.com/wp-content/uploads/2020/12/image-20.png)

## Workspaces:

**Rule of Thumb:** Use workspaces for environments that do not greatly deviate from one another, to avoid duplicating your configurations.

1. manage your workspaces in the CLI and be aware of the workspace you are working in to avoid accidentally performing operations on the wrong environment.
2. Your .tf file must be generic for both environments
3. You use the .tfvars file to apply the variables required for each environment

Now that your workspace handles the resources as individual environments, only one output is expected.

![](https://johnthompson365.com/wp-content/uploads/2020/12/image-21.png)

![](../.gitbook/assets/image%20%281%29.png)

## Create your own module

I want to be able to deploy a Single Node Windows or Linux Tableau server.

The simplest option seems to be using the directory structure so I can have **two separate .tf files** as there are some **different resources** required for each environment.


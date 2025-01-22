# Exercise 11: Provisioning a SAP BTP Landscape with Terraform

Welcome to **Exercise 11** of our hands-on series. In **Exercise 10**, you created a SAP BTP landscape manually via the SAP BTP Cockpit. That manual setup required multiple steps: creating subaccounts, enabling Cloud Foundry environments, assigning entitlements, creating service instances, and so on. Imagine you wanted four subaccounts (DEV, TEST, ACC, PROD) with multiple Cloud Foundry spaces, entitlements, service instances, and subscriptions—it quickly becomes a lot of repetitive work.

In this exercise, we will use **Terraform** to automate that entire setup. All you’ll need to do is to configure a set of files (like templates) and run a few Terraform commands. This is a one-time, local demonstration for learning purposes. In an actual production environment or for day-to-day management of an existing landscape, you would integrate Terraform into your CI/CD pipeline and store the state centrally. But that’s for a future discussion.

## Folder Structure

We are going to create all the necessary files inside a folder called `tf`. For now, imagine your project folder as follows:

```
.
└── tf
    ├── main.tf
    ├── variables.tf
    ├── provider.tf
    ├── account-info.vcs.auto.tfvars
    ├── dev.tf
    ├── acc.tf
    ├── modules
    │   └── sap_btp_entitlements
    │       ├── main.tf
    │       ├── variables.tf
    │       └── outputs.tf
    └── local-secrets.auto.tfvars (later in the exercises)
```

We’ll go through each file step by step. Create the files in the 'tf' folder!

Let’s start with a quick Terraform CLI overview.

---

## 1. Introduction to Terraform CLI

### What is Terraform?

[Terraform](https://www.terraform.io/) is an open-source Infrastructure as Code (IaC) tool by HashiCorp. It lets you define infrastructure in declarative configuration files, which you can version control and reuse. When you run Terraform commands, it compares your declared desired state with the actual state of your infrastructure and makes the necessary changes to reach the desired state.

### Why do we need it?

- **Consistency**: Instead of manually creating subaccounts and enabling services, you have everything in code.
- **Reusability**: You can share the same configuration to replicate the same environment in DEV, TEST, ACC, and PROD.
- **Transparency**: All changes are clear in source code and Terraform’s plan logs.

### Basic Terraform Commands

Here’s a quick overview of the most common commands:

- **`terraform -h`**  
  Displays help for the Terraform CLI, showing available subcommands and options.

- **`terraform init`**  
  Initializes your Terraform working directory, setting up local state (if needed), downloading providers, and preparing modules.

- **`terraform plan`**  
  Previews what actions Terraform will take (like a “dry run”). It compares your desired state in the `.tf` files to what is currently live and to what is in your local state file.

- **`terraform apply`**  
  Applies the changes to your infrastructure. After you confirm (by typing “yes”), Terraform will deploy or modify resources as needed.

- **`terraform fmt`**  
  Formats your `.tf` files to follow the canonical style and spacing rules.

**Exercise**:  
1. Open a terminal in your `tf` folder.  
2. Run `terraform -h` to see the help text.    

---

## 2. The `main.tf` File

Let’s look at a typical **`main.tf`** file for our scenario:

```terraform
terraform {
  required_version = ">= 1.4"

  required_providers {
    btp = {
      source  = "sap/btp"
      version = "1.7.0"
    }
    cloudfoundry = {
      source  = "cloudfoundry/cloudfoundry"
      version = "1.0.0"
    }
  }
}

resource "btp_directory" "parent" {
  name        = var.btp_settings.parent_directory
  description = var.btp_settings.parent_directory_description
  labels = {
    ManagedBy = ["Terraform"]
    Contact   = ["info@innov8ion.nl"]
  }
}
```

### Why is this file needed?

- **`terraform` block**: Specifies the minimum Terraform version to use (`>= 1.4`) and the required providers. Here we have two providers:
  - **`sap/btp`** for managing SAP BTP resources.
  - **`cloudfoundry/cloudfoundry`** for managing Cloud Foundry resources.
- **`resource "btp_directory" "parent"`**: Example resource that creates a **BTP Directory**, where you can later group your subaccounts.  
  - Uses variables from **`var.btp_settings`**.

**Exercise**:  
1. Create a `main.tf` file in your `tf` folder with the snippet above.  
2. Discuss or reflect on how the `required_providers` section helps Terraform know what dependencies it needs. Tip: have a look at the terraform providers (and their documentation): [Terraform BTP Provider](https://registry.terraform.io/providers/SAP/btp/latest) and can you find the right CloudFoundry provider? 

---

## 3. The `variables.tf` File

Terraform configurations can include variables to increase flexibility. Here’s an example **`variables.tf`**:

```terraform
###############################################################################################
# Variables set via local tfvars
###############################################################################################

# Settings for BTP platform
variable "btp_settings" {
  type = object({
    global_account               = string
    region                       = string
    api_url                      = string
    organisation                 = string
    parent_directory             = string
    parent_directory_description = string
  })
}

# Name for the resources
variable "YOUR_NAME" {
  type = string
}

# Username used to create btp resources
variable "BTP_USERNAME" {
  type = string
}

# Password used to create btp resources
variable "BTP_PASSWORD" {
  type      = string
  sensitive = true
}
```

### Why is this file needed?

- These variables define **all the parameters** we might need for our BTP landscape.
- **`type = object({...})`**: We group BTP-related settings in a single object for convenience.
- **`YOUR_NAME`, `BTP_USERNAME`, `BTP_PASSWORD`**: Individual variables that we’ll populate later via a `.tfvars` file or command-line arguments.

**Exercise**:  
1. Copy the above snippet to `variables.tf`.  
2. Reflect on how you could add or remove variables here if your scenario changes.

---

## 4. The `provider.tf` File

The **`provider.tf`** file configures how Terraform connects to each provider (SAP BTP, Cloud Foundry, etc.). Here’s an example:

```terraform
provider "btp" {
  globalaccount = var.btp_settings.global_account
  username      = var.BTP_USERNAME
  password      = var.BTP_PASSWORD
}

provider "cloudfoundry" {
  api_url  = var.btp_settings.api_url
  user     = var.BTP_USERNAME
  password = var.BTP_PASSWORD
}
```

### Why is this file needed?

- **Binds your variables** (from `variables.tf`) to the actual **provider** configurations.
- Tells Terraform: “To manage BTP resources, use `var.BTP_USERNAME` as the username, etc.”

**Exercise**:
1. Save this snippet as `provider.tf`.
2. Confirm your local variables from `variables.tf` match what the providers expect.

---

## 5. The `account-info.vcs.auto.tfvars` File

This file **`account-info.vcs.auto.tfvars`** automatically provides values for the variables. A snippet:

```terraform
btp_settings = {
  global_account               = "INNOV8iONBV"
  region                       = "eu10"
  api_url                      = "https://api.cf.eu10-005.hana.ondemand.com"
  organisation                 = "innov8ion"
  parent_directory             = <YOUR_NAME>
  parent_directory_description = "Parent directory for BTP resources managed by Terraform"
}
```

### Why is this file needed?

- **Stores default values** for the `btp_settings` object variable.
- Any file ending in `.auto.tfvars` is automatically loaded by Terraform.

> **Note**: Because the name includes `vcs` (version control system), you might treat this differently in real projects (deciding which secrets or environment-specific values to commit).  

**Exercise**:  
1. Create `account-info.vcs.auto.tfvars` in your `tf` folder with the snippet above.  
2. Adjust the values if needed (e.g., your own name).  

---

## 6. The `dev.tf` and `acc.tf` Files

Now we define the subaccounts for DEV and ACC. Below are two files: **`dev.tf`** and **`acc.tf`**. Each file is similar, except the name, subdomain, and environment references differ for DEV vs. ACC.

### `dev.tf`

```terraform
###############################################################################################
# Create a new sub account
###############################################################################################
resource "btp_subaccount" "subaccount_dev" {
  name        = "${var.YOUR_NAME}-dev"
  subdomain   = "i8-${var.YOUR_NAME}-dev"
  parent_id   = btp_directory.parent.id
  region      = var.btp_settings.region
  description = "My development subaccount"
  labels = {
    ManagedBy = ["Terraform"]
    Contact   = [var.YOUR_EMAIL]
  }
  usage = "NOT_USED_FOR_PRODUCTION"
}

###############################################################################################
# Add needed entitlements
###############################################################################################
module "sap_btp_entitlements_dev" {
  depends_on = [btp_subaccount.subaccount_dev]
  source     = "./modules/sap_btp_entitlements"

  subaccount = btp_subaccount.subaccount_dev.id

  entitlements = {
    "credstore" = ["free=1"]
  }
}

###############################################################################################
# Enable Cloud Foundry & create org and space
###############################################################################################
# Get the landscape label from the last environment
data "btp_subaccount_environments" "all_dev" {
  subaccount_id = btp_subaccount.subaccount_dev.id
}

resource "btp_subaccount_environment_instance" "cloudfoundry_dev" {
  depends_on       = [btp_subaccount.subaccount_dev, module.sap_btp_entitlements_dev]
  subaccount_id    = btp_subaccount.subaccount_dev.id
  name             = btp_subaccount.subaccount_dev.subdomain
  landscape_label  = data.btp_subaccount_environments.all_dev.values[length(data.btp_subaccount_environments.all_dev.values) - 1].landscape_label
  environment_type = "cloudfoundry"
  service_name     = "cloudfoundry"
  plan_name        = "standard"
  parameters = jsonencode({
    instance_name = btp_subaccount.subaccount_dev.subdomain
  })
}

resource "cloudfoundry_space" "space_dev" {
  depends_on = [btp_subaccount_environment_instance.cloudfoundry_dev]
  provider   = cloudfoundry
  name       = "dev"
  org        = btp_subaccount_environment_instance.cloudfoundry_dev.platform_id
  allow_ssh  = false
}

resource "cloudfoundry_space_role" "i_am_space_manager_dev" {
  depends_on = [cloudfoundry_space.space_dev]
  username   = var.YOUR_EMAIL
  type       = "space_manager"
  space      = cloudfoundry_space.space_dev.id
}

resource "cloudfoundry_space_role" "i_am_space_developer_dev" {
  depends_on = [cloudfoundry_space.space_dev]
  username   = var.YOUR_EMAIL
  type       = "space_developer"
  space      = cloudfoundry_space.space_dev.id
}

###############################################################################################
# Enable XSUAA API Access
###############################################################################################
data "cloudfoundry_service_plans" "xsuaa_dev" {
  depends_on            = [module.sap_btp_entitlements_dev, cloudfoundry_space_role.i_am_space_developer_dev, cloudfoundry_space_role.i_am_space_manager_dev]
  provider              = cloudfoundry
  service_offering_name = "xsuaa"
  name                  = "apiaccess"
}

resource "cloudfoundry_service_instance" "xsuaa_instance_dev" {
  provider     = cloudfoundry
  depends_on   = [data.cloudfoundry_service_plans.xsuaa_dev, cloudfoundry_space_role.i_am_space_developer_dev, cloudfoundry_space_role.i_am_space_manager_dev]
  name         = "xsuaa-apiaccess"
  type         = "managed"
  space        = cloudfoundry_space.space_dev.id
  service_plan = data.cloudfoundry_service_plans.xsuaa_dev.service_plans[0].id
}
```

### `acc.tf`

```terraform
###############################################################################################
# Create a new sub account
###############################################################################################
resource "btp_subaccount" "subaccount_acc" {
  name        = "${var.YOUR_NAME}-acc"
  subdomain   = "i8-${var.YOUR_NAME}-acc"
  parent_id   = btp_directory.parent.id
  region      = var.btp_settings.region
  description = "My acceptance subaccount"
  labels = {
    ManagedBy = ["Terraform"]
    Contact   = [var.YOUR_EMAIL]
  }
  usage = "NOT_USED_FOR_PRODUCTION"
}

###############################################################################################
# Add needed entitlements
###############################################################################################
module "sap_btp_entitlements_acc" {
  depends_on = [btp_subaccount.subaccount_acc]
  source     = "./modules/sap_btp_entitlements"

  subaccount = btp_subaccount.subaccount_acc.id

  entitlements = {
    "credstore" = ["free=1"]
  }
}

###############################################################################################
# Enable Cloud Foundry & create org and space
###############################################################################################
# Get the landscape label from the last environment
data "btp_subaccount_environments" "all_acc" {
  subaccount_id = btp_subaccount.subaccount_dev.id
}

resource "btp_subaccount_environment_instance" "cloudfoundry_acc" {
  depends_on       = [btp_subaccount.subaccount_acc, module.sap_btp_entitlements_acc]
  subaccount_id    = btp_subaccount.subaccount_acc.id
  name             = btp_subaccount.subaccount_acc.subdomain
  landscape_label  = data.btp_subaccount_environments.all_acc.values[length(data.btp_subaccount_environments.all_acc.values) - 1].landscape_label
  environment_type = "cloudfoundry"
  service_name     = "cloudfoundry"
  plan_name        = "standard"
  parameters = jsonencode({
    instance_name = btp_subaccount.subaccount_acc.subdomain
  })
}

resource "cloudfoundry_space" "space_acc" {
  depends_on = [btp_subaccount_environment_instance.cloudfoundry_acc]
  provider   = cloudfoundry
  name       = "acc"
  org        = btp_subaccount_environment_instance.cloudfoundry_acc.platform_id
  allow_ssh  = false
}

resource "cloudfoundry_space_role" "i_am_space_manager_acc" {
  depends_on = [cloudfoundry_space.space_acc]
  username   = var.YOUR_EMAIL
  type       = "space_manager"
  space      = cloudfoundry_space.space_acc.id
}

resource "cloudfoundry_space_role" "i_am_space_developer_acc" {
  depends_on = [cloudfoundry_space.space_acc]
  username   = var.YOUR_EMAIL
  type       = "space_developer"
  space      = cloudfoundry_space.space_acc.id
}

###############################################################################################
# Enable XSUAA API Access
###############################################################################################
data "cloudfoundry_service_plans" "xsuaa_acc" {
  depends_on            = [module.sap_btp_entitlements_acc, cloudfoundry_space_role.i_am_space_developer_dev, cloudfoundry_space_role.i_am_space_manager_dev]
  provider              = cloudfoundry
  service_offering_name = "xsuaa"
  name                  = "apiaccess"
}

resource "cloudfoundry_service_instance" "xsuaa_instance_acc" {
  provider     = cloudfoundry
  depends_on   = [data.cloudfoundry_service_plans.xsuaa_acc, cloudfoundry_space_role.i_am_space_developer_acc, cloudfoundry_space_role.i_am_space_manager_acc]
  name         = "xsuaa-apiaccess"
  type         = "managed"
  space        = cloudfoundry_space.space_acc.id
  service_plan = data.cloudfoundry_service_plans.xsuaa_acc.service_plans[0].id
}
```

### Why do we have these files?

- They define **subaccounts** (DEV, ACC) and the resources inside them (entitlements, Cloud Foundry environment, CF spaces, XSUAA).
- We’re using a **module** `sap_btp_entitlements` to handle entitlements (introduced in the next section).

> **Note**: Modules will be explained in the next exercise.

**Exercise**:
1. Create `dev.tf` and `acc.tf` with the snippets above.  
2. Inspect all that is happening, look at the previously used provider documentation to figure out what all this would do.
3. Observe the pattern: the ACC setup is almost the same as DEV but references `subaccount_acc` instead of `subaccount_dev`.  

---

## 7. Modules: `sap_btp_entitlements`

In `dev.tf` and `acc.tf`, we have references to the module `sap_btp_entitlements`:
```terraform
module "sap_btp_entitlements_dev" {
  ...
  source = "./modules/sap_btp_entitlements"
  ...
}
```
Here’s what that module looks like. It consists of three files in `modules/sap_btp_entitlements/`: **`main.tf`**, **`variables.tf`**, and **`outputs.tf`**.

### `main.tf`

```terraform
terraform {
  required_providers {
    btp = {
      source = "sap/btp"
    }
  }
}

locals {
  parsed_entitlements_data = flatten([
    for service, plans in var.entitlements : [
      for plan in plans : {
        service_name = service
        plan_name    = split("=", plan)[0]
        amount       = length(split("=", plan)) > 1 ? split("=", plan)[1] : "0"
      }
    ]
  ])
}

resource "btp_subaccount_entitlement" "entitlement" {
  for_each = {
    for entitlement in local.parsed_entitlements_data :
    "${entitlement.service_name}-${entitlement.plan_name}" => entitlement
  }
  subaccount_id = var.subaccount
  service_name  = each.value.service_name
  plan_name     = each.value.plan_name
  amount        = each.value.amount != "0" ? each.value.amount : null
}
```

### `variables.tf`

```terraform
variable "entitlements" {
  description = "Entitlements List"
  type        = map(list(string))
  default     = {}
  validation {
    condition = alltrue([
      for s, p in var.entitlements : length(p) == length(toset(p))
    ])
    error_message = "Each service inside 'entitlements' must have distinct plan names."
  }
}

variable "subaccount" {
  type = string
}
```

### `outputs.tf`

```terraform
output "subaccount" {
  value = var.subaccount
}

output "entitlement_list" {
  value = var.entitlements
}

output "parsed_entitlements" {
  value = local.parsed_entitlements_data
}

output "entitlements_created" {
  value = btp_subaccount_entitlement.entitlement
}
```

### Why do we have these files?

- **Modules** let you package and reuse Terraform configuration.  
- The `sap_btp_entitlements` module abstracts away the logic to create entitlements for a subaccount. In `dev.tf` and `acc.tf`, we simply provide a list of entitlements.
- The module’s `variables.tf` accepts an object that maps “service” → [“plan1”, …], and `main.tf` iterates over them to create the `btp_subaccount_entitlement` resources.
- The `outputs.tf` files can expose relevant data to other parts of your Terraform project.

**Exercise**:
1. Create the folder structure `modules/sap_btp_entitlements`.
2. Copy `main.tf`, `variables.tf`, and `outputs.tf` into it.
3. Notice how you can pass different entitlements from each environment (DEV, ACC, etc.) to the same module.

---

## 8. Running `terraform init`

After you’ve created all these files, run:

```
terraform init
```

What does this do?

- Downloads and installs the providers (in the `.terraform` folder).  
- Prepares modules (in this case, our local module references).  
- Creates or updates **`.terraform.lock.hcl`** to lock the provider versions used.

### .terraform Folder & .terraform.lock.hcl

- **`.terraform` folder**: Similar to a `node_modules` folder in Node.js—contains the downloaded provider binaries.  
- **`.terraform.lock.hcl`**: Similar to a `package-lock.json` in Node.js—locks the exact version of providers so subsequent runs are consistent.

**Exercise**:
1. Run `terraform init` in the `tf` folder.
2. Inspect the generated `.terraform` folder. (You’ll see subfolders for each provider.)
3. Look at `.terraform.lock.hcl`—it pins exact versions of the providers you declared.

---

## 9. Running `terraform plan`

Next, run:

```
terraform plan
```

This command will:

- Refresh your local state (if any) by default.  
- Compare the desired configuration in your `.tf` files with the current real-world state.  
- Propose any additions, changes, or deletions it needs to reconcile the differences.

You should see a list of resources that **Terraform** will create. At this point, you haven’t provided credentials or all variables yet, so make sure everything is set up (see next step for local secrets).

---

## 10. Simplifying Variable Inputs with `local-secrets.auto.tfvars`

You likely don’t want to provide all these variables on the CLI or type `-var="BTP_USERNAME=..." -var="BTP_PASSWORD=..." -var="YOUR_NAME=..."` every time. Let’s create a **`local-secrets.auto.tfvars`** file to store these values:

```terraform
BTP_USERNAME = "your.mail@mail.com"
BTP_PASSWORD = "password"
YOUR_NAME    = "yourname"
YOUR_EMAIL   = "your.mail@mail.com"
```

### Why does the `.auto.tfvars` extension matter?

- Terraform automatically loads `*.auto.tfvars` files.  
- If you keep the `auto` suffix, you don’t need to pass `-var-file=filename`.  
- If you remove `auto`, you must explicitly pass `-var-file=myvariables.tfvars` in `terraform plan` or `terraform apply`.

**Exercise**:
1. Create `local-secrets.auto.tfvars` (and do NOT commit it to version control if it contains real passwords).  
2. Try running `terraform plan` again, see if it picks up your variable values automatically.

---

## 11. Applying the Configuration

Finally, run:

```
terraform apply
```

Terraform will show the same plan it showed before. If you confirm (type “yes”), Terraform will start creating subaccounts, enabling environments, and so on in your SAP BTP account.

### The `terraform.tfstate` File

After a successful apply, Terraform creates (or updates) a **`terraform.tfstate`** file locally. It contains metadata about the resources you’ve created:

- **Not always the single source of truth**: If someone else modifies those resources outside of Terraform, your `.tfstate` will become out of sync. 
- In real-world usage, you keep the state file in a remote backend (like an S3 bucket or Terraform Cloud) so a team can share it, and you can do drift detection to ensure your declared state and the actual environment match.
- **`terraform refresh`** (which also happens automatically before a plan or apply) re-syncs the state with real-world resources—*but only for resources Terraform is already tracking*.

**Exercise**:
1. Run `terraform apply`.  
2. Review the subaccounts and resources created in the BTP Cockpit.  
3. Inspect `terraform.tfstate` to see how Terraform has recorded the resource metadata.
4. Can you add code to create an instance of the free credential store?

---

## Final Thoughts

- You’ve now learned how to create a basic multi-subaccount SAP BTP setup using Terraform.
- This demonstration was done locally. In real scenarios, you’d store the state in a remote backend and incorporate Terraform into a CI/CD pipeline.
- **Drift detection**: Keep an eye on changes outside of Terraform.  
- **Further expansions**: You can add more subaccounts, services, or spaces by adding more `.tf` files or reusing modules.

Congratulations—your SAP BTP environment is now set up with a single `terraform apply` command! In practice, you’d avoid manual modifications to keep the code as the single source of truth, set up version control, and possibly add automated validations in the pipeline. But for a local exercise, you’ve now seen the power of Infrastructure as Code for SAP BTP. Enjoy your automated BTP landscapes!
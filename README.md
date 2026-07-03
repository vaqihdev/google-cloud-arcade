# Provisioning Google Cloud Infrastructure with Terraform

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Google Cloud](https://img.shields.io/badge/GoogleCloud-%234285F4.svg?style=for-the-badge&logo=google-cloud&logoColor=white)
[![vaqihdev](https://img.shields.io/badge/%3E__vaqihdev:_%7E$-0f172a?style=for-the-badge)](https://vaqihdev.vercel.app/)

## 📌 Project Overview

This lab introduces the fundamentals of using Terraform as Infrastructure as Code (IaC) to manage Google Cloud resources. The entire infrastructure is created, modified, and destroyed using declarations in Terraform files without manual configuration via the Google Cloud Console.

### 📝 Cover

| Field | Value |
|--------|-------|
| **Project Name** | Terraform Fundamentals on Google Cloud |
| **Platform** | Google Cloud Skills Boost |
| **Category** | Infrastructure as Code (IaC) |
| **Technology** | Terraform, Google Cloud Platform |
| **Cloud Provider** | Google Cloud |
| **Author** | Muchamad Ghufron Vaqih |
| **Version** | 1.0 |
| **Created Date** | 2026-07-03 |
| **Last Updated** | 2026-07-03 |
| **Repository** | portfolio-learning |
| **Status** | Completed |
| **Tags** | Terraform, GCP, IaC, Compute Engine, VPC, Storage, Infrastructure Automation |

---

## 🎯 Objectives

- Understand the Terraform workflow.
- Initialize a Terraform project.
- Create a Virtual Private Cloud (VPC).
- Create a Compute Engine VM.
- Update resources using Terraform.
- Understand *destructive updates*.
- Use a Static External IP.
- Utilize resource *dependencies*.
- Use Provisioners.
- Manage Terraform State.
- Destroy the entire infrastructure.

---

## 🏗️ Architecture

```text
                Terraform
                    │
                    ▼
         Google Provider (GCP)
                    │
      ┌─────────────┴─────────────┐
      ▼                           ▼
  VPC Network              Static IP Address
      │                           │
      └──────────────┬────────────┘
                     ▼
            Compute Engine VM
                     │
              Local Provisioner
                     ▼
              ip_address.txt
```

---

## 🛠️ Technologies Used

- **Terraform** v1.15.7
- **Google Cloud Provider**
- **Google Compute Engine**
- **Google VPC**
- **Google Cloud Storage**
- **Cloud Shell**
- **Google Cloud Skills Boost**

---

## 🔄 Terraform Workflow

```text
Write Configuration
        │
        ▼
  terraform init
        │
        ▼
  terraform plan
        │
        ▼
  terraform apply
        │
        ▼
Infrastructure Created
        │
        ▼
 Modify Configuration
        │
        ▼
  terraform apply
        │
        ▼
 terraform destroy
```

---

## 📦 Resources Created

### Networking
- VPC Network
- Static External IP

### Compute
- Compute Engine VM
- Second Compute Engine VM

### Storage
- Cloud Storage Bucket

---

## 💻 Terraform Commands Used

| Command | Description |
|---------|-------------|
| `terraform init` | Initialize |
| `terraform validate` | Validate Configuration |
| `terraform fmt` | Format Configuration |
| `terraform plan` | View Execution Plan |
| `terraform plan -out static_ip` | Save Execution Plan |
| `terraform apply` | Apply Infrastructure |
| `terraform apply static_ip` | Apply Saved Plan |
| `terraform show` | Show Infrastructure |
| `terraform destroy` | Destroy Infrastructure |
| `terraform taint google_compute_instance.vm_instance` | Force Recreation |

---

## 🚀 Implementation Breakdown

### Stage 1 — Initialize the Terraform Project
**Objective:** Initialize the Terraform project and download the required Google Cloud provider plugins.

**Terraform Configuration:**
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "3.5.0"
    }
  }
}

provider "google" {
  project = "PROJECT_ID"
  region  = "REGION"
  zone    = "ZONE"
}

resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
```

**Commands:**
```bash
terraform init
```

**Expected Output:**
> Terraform has been successfully initialized.

**Explanation:**
- Downloads the required provider plugins.
- Creates the `.terraform` directory.
- Generates `.terraform.lock.hcl`.
- Initializes the Terraform working directory.

---

### Stage 2 — Create a Virtual Private Cloud (VPC)
**Objective:** Provision a custom Virtual Private Cloud on Google Cloud.

**Terraform Configuration:**
```hcl
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
```

**Commands:**
```bash
terraform plan
terraform apply
```

**Expected Output:**
> `google_compute_network.vpc_network`: Creation complete

**Explanation:**
- Terraform provisions a new VPC network named `terraform-network`.

---

### Stage 3 — Deploy a Compute Engine Instance
**Objective:** Provision a virtual machine inside the VPC network.

**Terraform Configuration:**
```hcl
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "e2-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {}
  }
}
```

**Commands:**
```bash
terraform plan
terraform apply
```

**Expected Output:**
> `google_compute_instance.vm_instance`: Creation complete

**Explanation:**
Terraform creates a Compute Engine instance using:
- Debian 12
- `e2-micro` machine type
- Ephemeral external IP address

---

### Stage 4 — Update Network Tags
**Objective:** Assign network tags to the virtual machine.

**Terraform Configuration:**
```hcl
tags = [
  "web",
  "dev"
]
```

**Commands:**
```bash
terraform apply
```

**Explanation:**
- Terraform updates the instance metadata without replacing the VM.

---

### Stage 5 — Perform a Destructive Update
**Objective:** Replace the operating system image.

**Previous Configuration:**
```hcl
image = "debian-cloud/debian-12"
```

**Updated Configuration:**
```hcl
image = "cos-cloud/cos-stable"
```

**Commands:**
```bash
terraform apply
```

**Expected Output:**
> Destroying...  
> Creating...  
> Apply complete.

**Explanation:**
- Changing the boot image requires the virtual machine to be destroyed and recreated because the image attribute is immutable.

---

### Stage 6 — Configure a Static External IP Address
**Objective:** Assign a static external IP address to the Compute Engine instance to ensure the public IP remains unchanged after instance recreation.

**Terraform Configuration:**

Create a static IP resource:
```hcl
resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}
```

Update the network interface:
```hcl
network_interface {
  network = google_compute_network.vpc_network.self_link

  access_config {
    nat_ip = google_compute_address.vm_static_ip.address
  }
}
```

**Commands:**
```bash
terraform plan -out static_ip
terraform apply static_ip
```

**Expected Output:**
> Plan: 1 to add, 1 to change, 0 to destroy.  
> Apply complete!

**Explanation:**
- Terraform creates a new Static External IP resource and associates it with the existing Compute Engine instance.
- Unlike ephemeral IP addresses, the assigned IP address remains consistent even if the virtual machine is restarted.

**Skills Acquired:** Google Compute Address, Static Public Networking

---

### Stage 7 — Create a Cloud Storage Bucket
**Objective:** Provision a Cloud Storage bucket using Terraform.

**Terraform Configuration:**
```hcl
resource "google_storage_bucket" "example_bucket" {
  name     = "your-unique-bucket-name"
  location = "US"

  website {
    main_page_suffix = "index.html"
    not_found_page   = "404.html"
  }
}
```

**Commands:**
```bash
terraform apply
```

**Expected Output:**
> `google_storage_bucket.example_bucket`: Creation complete

**Explanation:**
- Terraform provisions a globally unique Cloud Storage bucket.
- Since bucket names are globally unique across Google Cloud, a unique naming convention is required.

**Skills Acquired:** Google Cloud Storage, Global Resource Naming, Storage Resource Provisioning

---

### Stage 8 — Create a Second Compute Engine Instance
**Objective:** Deploy another Compute Engine instance that explicitly depends on the Storage Bucket.

**Terraform Configuration:**
```hcl
resource "google_compute_instance" "another_instance" {
  depends_on = [
    google_storage_bucket.example_bucket
  ]

  name         = "terraform-instance-2"
  machine_type = "e2-micro"

  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {}
  }
}
```

**Commands:**
```bash
terraform apply
```

**Expected Output:**
> `google_storage_bucket.example_bucket`: Creation complete  
> `google_compute_instance.another_instance`: Creation complete

**Explanation:**
- Although Terraform automatically detects many dependencies, the `depends_on` argument explicitly guarantees that the Cloud Storage bucket is created before the second virtual machine.

**Skills Acquired:** Explicit Dependency Management, Resource Ordering, Terraform Dependency Graph

---

### Stage 9 — Remove Resources
**Objective:** Remove the Cloud Storage bucket and the second virtual machine from the Terraform configuration.

**Actions:**
Delete the following resources from `main.tf`:
- `google_storage_bucket.example_bucket`
- `google_compute_instance.another_instance`

**Commands:**
```bash
terraform apply
```

**Expected Output:**
> Plan: 0 to add, 0 to change, 2 to destroy.  
> Destroy complete.

**Explanation:**
- Terraform compares the current state with the desired configuration and removes resources that no longer exist in the configuration file.

**Skills Acquired:** Infrastructure Drift Management, Resource Deletion, Desired State Reconciliation

---

### Stage 10 — Execute a Local Provisioner
**Objective:** Execute a local command immediately after the virtual machine is successfully created.

**Terraform Configuration:**
```hcl
provisioner "local-exec" {
  command = "echo Instance ${self.name} has IP ${self.network_interface[0].access_config[0].nat_ip} >> ip_address.txt"
}
```

**Commands:**
```bash
terraform apply
```

**Explanation:**
- Since the resource already exists, Terraform detects no infrastructure changes. Therefore, the provisioner is not executed.

---

### Stage 11 — Force Resource Recreation
**Objective:** Force Terraform to recreate the virtual machine and execute the provisioner.

**Commands:**
```bash
terraform taint google_compute_instance.vm_instance
terraform apply
```

**Expected Output:**
> `google_compute_instance.vm_instance`: Destroying...  
> `google_compute_instance.vm_instance`: Creating...  
> Apply complete!

**Explanation:**
- The `terraform taint` command marks the resource as damaged.
- During the next `terraform apply`, Terraform destroys and recreates the instance, triggering the `local-exec` provisioner.

**Generated File:** `ip_address.txt`  
**Contents:**
> Instance terraform-instance has IP xxx.xxx.xxx.xxx

**Skills Acquired:** Terraform Taint, Local Provisioners, Resource Recreation, Post-Provision Automation

---

### Stage 12 — Destroy the Entire Infrastructure
**Objective:** Remove every resource managed by Terraform.

**Commands:**
```bash
terraform destroy
```

**Expected Output:**
> Destroy complete! Resources: X destroyed.

**Resources Removed:**
- Compute Engine Instance
- Virtual Private Cloud
- Static External IP
- Cloud Storage Bucket

**Explanation:**
- Terraform compares the infrastructure state with the destroy operation and safely removes every managed resource.
- This step ensures no unnecessary cloud resources remain after completing the lab.

**Skills Acquired:** Infrastructure Cleanup, Cost Optimization, Terraform State Management, Resource Lifecycle Management

---

## 💡 Key Terraform Concepts Learned

| Concept | Description | Example |
|---------|-------------|---------|
| **Provider** | Defines which cloud platform Terraform manages. | `provider "google" {}` |
| **Resource** | Represents an infrastructure object. | Compute Engine, VPC, Storage Bucket |
| **State** | Stores the current infrastructure state to calculate changes. | `terraform.tfstate` |
| **Plan** | Shows infrastructure changes before execution. | `terraform plan` |
| **Apply** | Applies desired configuration. | `terraform apply` |
| **Destroy** | Deletes managed infrastructure. | `terraform destroy` |
| **Dependency** | Automatic or explicit ordering of resource creation. | `depends_on = [google_storage_bucket.bucket]` |
| **Provisioner** | Executes local/remote commands after resource creation. | `provisioner "local-exec" { ... }` |

---

## ⚠️ Challenges Encountered

### 1. Terraform Binary Conflict
- **Problem:** Cloud Shell executed `/google/bin/terraform` instead of the installed HashiCorp Terraform binary.
- **Root Cause:** Google wrapper binary was cached in PATH.
- **Resolution:** Executed Terraform using absolute path:
  ```bash
  /usr/bin/terraform
  ```

### 2. Invalid Bucket Name
- **Problem:** `Invalid bucket name` error.
- **Root Cause:** Placeholder bucket name was not replaced.
- **Resolution:** Used a globally unique bucket name.

### 3. Placeholder Configuration
- **Problem:** Using `PROJECT_ID`, `ZONE`, `REGION` directly caused permission errors.
- **Resolution:** Replaced placeholders with actual lab project values.

### 4. Dependency Cycle
- **Problem:** Terraform returned a `Cycle` error.
- **Root Cause:** Provisioner referenced the resource directly (`google_compute_instance.vm_instance`).
- **Resolution:** Used `self.name` instead of the full resource reference inside the provisioner.

---

## 🌟 Best Practices Learned

- [x] Always run `terraform fmt` to format code consistently.
- [x] Validate configuration with `terraform validate` before applying.
- [x] Always review execution plans (`terraform plan`).
- [x] Use explicit dependencies (`depends_on`) only when necessary.
- [x] Avoid hardcoding temporary values.
- [x] Use unique Cloud Storage bucket names.
- [x] Destroy lab resources after completion to avoid unwanted charges.
- [x] Keep Terraform configuration under version control.

---

## 🏆 Skills Gained

- Infrastructure as Code (IaC)
- Terraform CLI
- Google Cloud Provider
- Resource Lifecycle Management
- Dependency Management
- Static IP Configuration
- Compute Engine Deployment
- Provisioners
- Terraform State Management
- Infrastructure Automation

---

## 📁 Repository Structure

```text
terraform-fundamentals/
├── README.md
├── main.tf
├── terraform.tfstate
├── terraform.tfstate.backup
├── .terraform/
├── .terraform.lock.hcl
└── ip_address.txt
```

---

## ✅ Completion Status

**Completed Successfully** 🎉

---

## 📚 References

- [Terraform Documentation](https://developer.hashicorp.com/terraform)
- [Terraform Google Cloud Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [Google Cloud Documentation](https://cloud.google.com/docs)
- [Compute Engine Documentation](https://cloud.google.com/compute/docs)
- [Cloud Storage Documentation](https://cloud.google.com/storage/docs)
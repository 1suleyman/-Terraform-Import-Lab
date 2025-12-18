# 🧩 Terraform Import Lab 

In this lab, you learned how to **import existing AWS resources** (created *outside* of Terraform) into Terraform state so that Terraform can manage them going forward. This included identifying unmanaged resources, using the AWS CLI to inspect instances, creating matching Terraform resource blocks, and finally importing the resource with `terraform import`.

---

## 📋 Lab Overview

**Goal:**

* Inspect existing AWS resources created outside Terraform
* Identify unmanaged EC2 instances
* Use `terraform import` to bring an external instance under Terraform management
* Understand why imported resources must have full configuration blocks
* Complete resource configuration by matching the imported state

**Learning Outcomes:**

* Difference between resources in **AWS** vs **Terraform state**
* How to use `aws ec2 describe-instances` with `--endpoint`
* How to create placeholder resource blocks for import
* How to run `terraform import` correctly
* How to inspect imported attributes and translate them into Terraform code

---

## 🛠 Step-by-Step Journey

### Step 1: Navigate to the Project Jade Directory

Directory:

```
/root/terraform-projects/project-jade
```

Inspect existing resources using:

* `main.tf`
* `variables.tf`
* `terraform.tfstate`

---

### Step 2: Identify Which Resource Is Not Part of Terraform

Correct answer:

➡️ **The EC2 instance named `jade-mw` is NOT part of the Terraform configuration.**

Terraform only manages what is defined inside the `.tf` files AND appears in state.

---

### Step 3: Identify the SSH Key Used by the Instances

Answer:

➡️ **Key name: `jade`**

Confirmed inside `variables.tf`.

---

### Step 4: Is the Key Created by Terraform?

Answer:

❌ **No.**

The key `jade` was created using the AWS CLI.

---

### Step 5: The CLI Command Used to Create the Key

The lab confirms that `jade.pem` was created using:

```bash
aws ec2 create-key-pair \
  --endpoint http://aws:4566 \
  --key-name jade \
  --query "KeyMaterial" \
  --output text > /root/terraform-projects/project-jade/jade.pem
```

✔️ This key was generated outside Terraform
✔️ Must NOT be recreated by Terraform

---

### Step 6: Inspect EC2 Instance `jade-mw` Created Outside Terraform

Retrieve instance details:

```bash
aws ec2 describe-instances \
  --endpoint http://aws:4566
```

Instance ID found:

➡️ **`i-1ecc21…`**

AMI, type, and key also match:

* AMI: `ami-082…`
* Instance type: `t2.large`
* Key name: `jade`

---

### Step 7: Create a Resource Block for Import

In `main.tf`:

```hcl
resource "aws_instance" "jade-mw" {

}
```

✔️ Empty block required before import
✔️ Terraform must know *where* to store the imported state

---

### Step 8: Import the Instance Into Terraform State

Ensure you're in the correct folder:

```bash
cd /root/terraform-projects/project-jade
```

Then import:

```bash
terraform import aws_instance.jade-mw i-1ecc21...
```

Terraform outputs:

✔️ **Import successful**

---

### Step 9: What Happens if You Run `terraform apply` Now?

Answer:

❌ **It will error — "resource arguments not defined".**

Reason:

The imported resource has **state**, but the configuration block is **empty**, so Terraform cannot reconcile desired state vs actual state.

---

### Step 10: Complete the Resource Block Using the Imported Attributes

Use:

```bash
terraform show
```

or:

```bash
terraform show -json | jq
```

Or AWS CLI:

```bash
aws ec2 describe-instances --endpoint http://aws:4566
```

Fill in the attributes:

```hcl
resource "aws_instance" "jade-mw" {
  ami           = "ami-082b3..."         # from state
  instance_type = "t2.large"
  key_name      = "jade"

  tags = {
    Name = "jade-mw"
  }

  timeouts {
    create = "10m"
    delete = "10m"
  }
}
```

✔️ Matches state
✔️ No drift
✔️ Terraform can now manage the instance safely

Apply:

```bash
terraform apply
```

Terraform should output **no changes**.

---

## ✅ Key Commands Summary

| Purpose                       | Command                                                 |     |
| ----------------------------- | ------------------------------------------------------- | --- |
| Describe EC2 instances        | `aws ec2 describe-instances --endpoint http://aws:4566` |     |
| Create placeholder resource   | `resource "aws_instance" "name" {}`                     |     |
| Import a resource             | `terraform import aws_instance.name <instance-id>`      |     |
| Show full state               | `terraform show`                                        |     |
| Show JSON state for debugging | `terraform show -json                                   | jq` |
| Apply configuration           | `terraform apply`                                       |     |

---

## 💡 Notes / Tips

* Terraform **cannot** create a resource during import — the resource must already exist.
* Every argument used when the resource was created outside Terraform **must be added manually**.
* Imported resources do **not** include lifecycle rules, provisioners, or user data — you must add them if needed.
* Running `terraform apply` before completing the configuration will cause **errors** or **resource replacement**.
* Always compare imported attributes against expected Terraform code to avoid drift.

---

## 📌 Lab Summary

| Step                        | Status | Key Takeaway                        |
| --------------------------- | ------ | ----------------------------------- |
| Identify unmanaged instance | ✅      | `jade-mw` created outside Terraform |
| Inspect EC2 instance        | ✅      | CLI used with `--endpoint`          |
| Create placeholder resource | ✅      | Required before import              |
| Import instance             | ✅      | Terraform state updated             |
| Complete resource block     | ✅      | Prevents errors & drift             |
| Final apply                 | ✅      | No changes required                 |

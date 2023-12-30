---
layout: post
title: Using YAML Data In Terraform
---

The most common way to pass data into Terraform is through variables. But what if you have a larger dataset that would be more easily maintained using a common data storage format? Or maybe you want to manipulate the data programatically? Storing the data in YAML rather than variables provides for that flexibility.

First, let's look at an example of storing data in a variable.

---

Say I have a list of employees that I need to create IAM users for.

I could store the employees' data in a map variable like this:
```terraform
variable "employees" {
  type = map(any)
  default = {
    "Alexander" = {
      "department" = "Engineering"
      "title"      = "Developer"
    }
    "Benjamin" = {
      "department" = "Finance"
      "title"      = "Accountant"
    }
    "Elijah" = {
      "department" = "Engineering"
      "title"      = "Director"
    }
    "Olivia" = {
      "department" = "Executive"
      "title"      = "CEO"
    }
    "Sophia" = {
      "department" = "Human Resources"
      "title"      = "Manager"
    }
  }
}
```

I'll use Terraform's [`for_each`](https://developer.hashicorp.com/terraform/language/meta-arguments/for_each) to loop over the data in the `employees` variable and create IAM users for each employee:
```terraform
resource "aws_iam_user" "users" {
  for_each = var.employees
  name     = each.key
  tags     = each.value
}
```

However, what if I wanted to store the data in a more manageable way? Or if I wanted to easily be able to update the data programmatically? Let's store it as YAML!

---

Here, I've moved the data to a file called `employees.yaml`:
```yaml
---
Alexander:
  title: Developer
  department: Engineering
Olivia:
  title: CEO
  department: Executive
Benjamin:
  title: Accountant
  department: Finance
Sophia:
  title: Manager
  department: Human Resources
Elijah:
  title: Director
  department: Engineering
```

Terraform's [`yamldecode`](https://developer.hashicorp.com/terraform/language/functions/yamldecode) function takes YAML as input and translates it to an appropriate Terraform data type.

Let's see what `employees.yaml` looks like when processed by `yamldecode` in `terraform console`:
```
> yamldecode(file("${path.module}/employees.yaml"))
{
  "Alexander" = {
    "department" = "Engineering"
    "title" = "Developer"
  }
  "Benjamin" = {
    "department" = "Finance"
    "title" = "Accountant"
  }
  "Elijah" = {
    "department" = "Engineering"
    "title" = "Director"
  }
  "Olivia" = {
    "department" = "Executive"
    "title" = "CEO"
  }
  "Sophia" = {
    "department" = "Human Resources"
    "title" = "Manager"
  }
}
```
It's exactly the same as what the `employees` variable was set to previously!

Now, I just need to modify the `aws_iam_user` resource to use the YAML file rather than the `employees` variable:
```terraform
resource "aws_iam_user" "users" {
  for_each = yamldecode(file("${path.module}/employees.yaml"))
  name     = each.key
  tags     = each.value
}
```

Finally here's the output of `terraform plan`:
```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_iam_user.users["Alexander"] will be created
  + resource "aws_iam_user" "users" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "Alexander"
      + path          = "/"
      + tags          = {
          + "department" = "Engineering"
          + "title"      = "Developer"
        }
      + tags_all      = {
          + "department" = "Engineering"
          + "title"      = "Developer"
        }
      + unique_id     = (known after apply)
    }

  # aws_iam_user.users["Benjamin"] will be created
  + resource "aws_iam_user" "users" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "Benjamin"
      + path          = "/"
      + tags          = {
          + "department" = "Finance"
          + "title"      = "Accountant"
        }
      + tags_all      = {
          + "department" = "Finance"
          + "title"      = "Accountant"
        }
      + unique_id     = (known after apply)
    }

  # aws_iam_user.users["Elijah"] will be created
  + resource "aws_iam_user" "users" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "Elijah"
      + path          = "/"
      + tags          = {
          + "department" = "Engineering"
          + "title"      = "Director"
        }
      + tags_all      = {
          + "department" = "Engineering"
          + "title"      = "Director"
        }
      + unique_id     = (known after apply)
    }

  # aws_iam_user.users["Olivia"] will be created
  + resource "aws_iam_user" "users" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "Olivia"
      + path          = "/"
      + tags          = {
          + "department" = "Executive"
          + "title"      = "CEO"
        }
      + tags_all      = {
          + "department" = "Executive"
          + "title"      = "CEO"
        }
      + unique_id     = (known after apply)
    }

  # aws_iam_user.users["Sophia"] will be created
  + resource "aws_iam_user" "users" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "Sophia"
      + path          = "/"
      + tags          = {
          + "department" = "Human Resources"
          + "title"      = "Manager"
        }
      + tags_all      = {
          + "department" = "Human Resources"
          + "title"      = "Manager"
        }
      + unique_id     = (known after apply)
    }

Plan: 5 to add, 0 to change, 0 to destroy.
```

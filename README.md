<p align="center"> <img src="https://user-images.githubusercontent.com/50652676/62349836-882fef80-b51e-11e9-99e3-7b974309c7e3.png" width="100" height="100"></p>


<h1 align="center">
    Terraform AWS KMS
</h1>

<p align="center" style="font-size: 1.2rem;">
    Terraform module to create vpc-peering resource on AWS.
     </p>

<p align="center">

<a href="https://www.terraform.io">
  <img src="https://img.shields.io/badge/Terraform-v1.7.0-green" alt="Terraform">
</a>
<a href="https://github.com/slovink/terraform-aws-vpc-peering/blob/main/LICENSE">
  <img src="https://img.shields.io/badge/License-APACHE-blue.svg" alt="Licence">
</a>



</p>
<p align="center">

<a href='https://www.facebook.com/Slovink.in=https://github.com/slovink/terraform-vpc-peering'>
  <img title="Share on Facebook" src="https://user-images.githubusercontent.com/50652676/62817743-4f64cb80-bb59-11e9-90c7-b057252ded50.png" />
</a>
<a href='https://www.linkedin.com/company/101534993/admin/feed/posts/=https://github.com/slovink/terraform-vpc-peering'>
  <img title="Share on LinkedIn" src="https://user-images.githubusercontent.com/50652676/62817742-4e339e80-bb59-11e9-87b9-a1f68cae1049.png" />
</a>



- [Introduction](#introduction)
- [Usage](#usage)
- [Module Inputs](#module-inputs)
- [Module Outputs](#module-outputs)
- [Examples](#examples)
- [License](#license)


## Prerequisites

This module has a few dependencies:

- [Terraform 1.x.x](https://learn.hashicorp.com/terraform/getting-started/install.html)
- [Go](https://golang.org/doc/install)







## Examples

**IMPORTANT:** Since the `master` branch used in `source` varies based on new modifications, we suggest that you use the release versions [here](https://github.com/slovink/terraform-aws-kms/tree/vinod/_example).

## License
This Terraform module is provided under the '[License Name]' License. Please see the [LICENSE](https://github.com/slovink/terraform-aws-kms/blob/vinod/LICENSE) file for more details.

### default Example
Here is an example of how you can use this module in your inventory structure:
  ```hcl
    module "kms_key" {
        source                  = "./../../"
        name                    = "kms"
        environment             = "test"
        deletion_window_in_days = 7
        alias                   = "alias/cloudtrail_Name"
        kms_key_enabled         = true
        multi_region            = true
        valid_to                = "2023-11-21T23:20:50Z"
        policy                  = data.aws_iam_policy_document.default.json
        }

        data "aws_caller_identity" "current" {}
        data "aws_partition" "current" {}

        data "aws_iam_policy_document" "default" {
        version = "2012-10-17"
        statement {
        sid    = "Enable IAM User Permissions"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions   = ["kms:*"]
        resources = ["*"]
        }
        statement {
        sid    = "Allow CloudTrail to encrypt logs"
        effect = "Allow"
        principals {
          type        = "Service"
          identifiers = ["cloudtrail.amazonaws.com"]
        }
        actions   = ["kms:GenerateDataKey*"]
        resources = ["*"]
        condition {
          test     = "StringLike"
          variable = "kms:EncryptionContext:aws:cloudtrail:arn"
          values   = ["arn:aws:cloudtrail:*:XXXXXXXXXXXX:trail/*"]
        }
        }

        statement {
        sid    = "Allow CloudTrail to describe key"
        effect = "Allow"
        principals {
          type        = "Service"
          identifiers = ["cloudtrail.amazonaws.com"]
        }
        actions   = ["kms:DescribeKey"]
        resources = ["*"]
        }

        statement {
        sid    = "Allow principals in the account to decrypt log files"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions = [
          "kms:Decrypt",
          "kms:ReEncryptFrom"
        ]
        resources = ["*"]
        condition {
          test     = "StringEquals"
          variable = "kms:CallerAccount"
          values = [
            "XXXXXXXXXXXX"]
        }
        condition {
          test     = "StringLike"
          variable = "kms:EncryptionContext:aws:cloudtrail:arn"
          values   = ["arn:aws:cloudtrail:*:XXXXXXXXXXXX:trail/*"]
        }
        }

        statement {
        sid    = "Allow alias creation during setup"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions   = ["kms:CreateAlias"]
        resources = ["*"]
        }
    }

  ```

### kms-key-external Example

  ```hcl
    module "kms_key" {
      source                  = "./../../"
      name                    = "kms"
      environment             = "test"
      deletion_window_in_days = 7
      alias                   = "alias/external_key"
      kms_key_enabled         = false
      enabled                 = true
      multi_region            = true
      create_external_enabled = true
      valid_to                = "2023-11-21T23:20:50Z"
      key_material_base64     = "WblXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
      policy                  = data.aws_iam_policy_document.default.json
     }

     data "aws_caller_identity" "current" {}
     data "aws_partition" "current" {}

     data "aws_iam_policy_document" "default" {
      version = "2012-10-17"
      statement {
        sid    = "Enable IAM User Permissions"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions   = ["kms:*"]
        resources = ["*"]
      }
      statement {
        sid    = "Allow CloudTrail to encrypt logs"
        effect = "Allow"
        principals {
          type        = "Service"
          identifiers = ["cloudtrail.amazonaws.com"]
        }
        actions   = ["kms:GenerateDataKey*"]
        resources = ["*"]
        condition {
          test     = "StringLike"
          variable = "kms:EncryptionContext:aws:cloudtrail:arn"
          values   = ["arn:aws:cloudtrail:*:XXXXXXXXXXXX:trail/*"]
        }
      }

      statement {
        sid    = "Allow CloudTrail to describe key"
        effect = "Allow"
        principals {
          type        = "Service"
          identifiers = ["cloudtrail.amazonaws.com"]
        }
        actions   = ["kms:DescribeKey"]
        resources = ["*"]
      }

      statement {
        sid    = "Allow principals in the account to decrypt log files"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions = [
          "kms:Decrypt",
          "kms:ReEncryptFrom"
        ]
        resources = ["*"]
        condition {
          test     = "StringEquals"
          variable = "kms:CallerAccount"
          values = [
          "XXXXXXXXXXXX"]
        }
        condition {
          test     = "StringLike"
          variable = "kms:EncryptionContext:aws:cloudtrail:arn"
          values   = ["arn:aws:cloudtrail:*:XXXXXXXXXXXX:trail/*"]
        }
      }

      statement {
        sid    = "Allow alias creation during setup"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions   = ["kms:CreateAlias"]
        resources = ["*"]
      }
    }
  ```
### kms-key-replica Example

  ```hcl
      module "kms_key" {
      source                  = "./../../"
      name                    = "kms"
      environment             = "test"
      deletion_window_in_days = 7
      alias                   = "alias/replicate_key"
      kms_key_enabled         = false
      create_replica_enabled  = true
      enabled                 = true
      multi_region            = false
      primary_key_arn         = "arn:aws:kms:xxxxxxxxxxxxxxxxxxxxx"
      policy                  = data.aws_iam_policy_document.default.json
    }

    data "aws_caller_identity" "current" {}
    data "aws_partition" "current" {}

    data "aws_iam_policy_document" "default" {
      version = "2012-10-17"
      statement {
        sid    = "Enable IAM User Permissions"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions   = ["kms:*"]
        resources = ["*"]
      }
      statement {
        sid    = "Allow CloudTrail to encrypt logs"
        effect = "Allow"
        principals {
          type        = "Service"
          identifiers = ["cloudtrail.amazonaws.com"]
        }
        actions   = ["kms:GenerateDataKey*"]
        resources = ["*"]
        condition {
          test     = "StringLike"
          variable = "kms:EncryptionContext:aws:cloudtrail:arn"
          values   = ["arn:aws:cloudtrail:*:XXXXXXXXXXXX:trail/*"]
        }
      }

      statement {
        sid    = "Allow CloudTrail to describe key"
        effect = "Allow"
        principals {
          type        = "Service"
          identifiers = ["cloudtrail.amazonaws.com"]
        }
        actions   = ["kms:DescribeKey"]
        resources = ["*"]
      }

      statement {
        sid    = "Allow principals in the account to decrypt log files"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions = [
          "kms:Decrypt",
          "kms:ReEncryptFrom"
        ]
        resources = ["*"]
        condition {
          test     = "StringEquals"
          variable = "kms:CallerAccount"
          values = [
          "XXXXXXXXXXXX"]
        }
        condition {
          test     = "StringLike"
          variable = "kms:EncryptionContext:aws:cloudtrail:arn"
          values   = ["arn:aws:cloudtrail:*:XXXXXXXXXXXX:trail/*"]
        }
      }

      statement {
        sid    = "Allow alias creation during setup"
        effect = "Allow"
        principals {
          type = "AWS"
          identifiers = [
            format(
              "arn:%s:iam::%s:root",
              join("", data.aws_partition.current[*].partition),
              data.aws_caller_identity.current.account_id
            )
          ]
        }
        actions   = ["kms:CreateAlias"]
        resources = ["*"]
      }
    }
  ```


## Feedback
If you come accross a bug or have any feedback, please log it in our [issue tracker](https://github.com/slovink/terraform-aws-kms.git), or feel free to drop us an email at [devops@slovink.com](devops@slovink.com).

If you have found it worth your time, go ahead and give us a ★ on [our GitHub](https://github.com/slovink/terraform-aws-kms.git)!
<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.6.4, < 1.7.0 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 5.32.1 |
| <a name="requirement_tls"></a> [tls](#requirement\_tls) | >= 3.0.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 5.32.1 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_labels"></a> [labels](#module\_labels) | git::git@github.com:slovink/terraform-aws-labels.git | vinod |

## Resources

| Name | Type |
|------|------|
| [aws_kms_alias.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/kms_alias) | resource |
| [aws_kms_external_key.external](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/kms_external_key) | resource |
| [aws_kms_key.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/kms_key) | resource |
| [aws_kms_replica_external_key.replica_external](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/kms_replica_external_key) | resource |
| [aws_kms_replica_key.replica](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/kms_replica_key) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_alias"></a> [alias](#input\_alias) | The display name of the alias. The name must start with the word `alias` followed by a forward slash. | `string` | `""` | no |
| <a name="input_bypass_policy_lockout_safety_check"></a> [bypass\_policy\_lockout\_safety\_check](#input\_bypass\_policy\_lockout\_safety\_check) | A flag to indicate whether to bypass the key policy lockout safety check. Setting this value to true increases the risk that the KMS key becomes unmanageable | `bool` | `false` | no |
| <a name="input_create_external_enabled"></a> [create\_external\_enabled](#input\_create\_external\_enabled) | Determines whether an external CMK (externally provided material) will be created or a standard CMK (AWS provided material) | `bool` | `false` | no |
| <a name="input_create_replica_enabled"></a> [create\_replica\_enabled](#input\_create\_replica\_enabled) | Determines whether a replica standard CMK will be created (AWS provided material) | `bool` | `false` | no |
| <a name="input_create_replica_external_enabled"></a> [create\_replica\_external\_enabled](#input\_create\_replica\_external\_enabled) | Determines whether a replica external CMK will be created (externally provided material) | `bool` | `false` | no |
| <a name="input_customer_master_key_spec"></a> [customer\_master\_key\_spec](#input\_customer\_master\_key\_spec) | Specifies whether the key contains a symmetric key or an asymmetric key pair and the encryption algorithms or signing algorithms that the key supports. Valid values: SYMMETRIC\_DEFAULT, RSA\_2048, RSA\_3072, RSA\_4096, ECC\_NIST\_P256, ECC\_NIST\_P384, ECC\_NIST\_P521, or ECC\_SECG\_P256K1. Defaults to SYMMETRIC\_DEFAULT. | `string` | `"SYMMETRIC_DEFAULT"` | no |
| <a name="input_deletion_window_in_days"></a> [deletion\_window\_in\_days](#input\_deletion\_window\_in\_days) | Duration in days after which the key is deleted after destruction of the resource. | `number` | `10` | no |
| <a name="input_description"></a> [description](#input\_description) | The description of the key as viewed in AWS console. | `string` | `"Parameter Store KMS master key"` | no |
| <a name="input_enable_key_rotation"></a> [enable\_key\_rotation](#input\_enable\_key\_rotation) | Specifies whether key rotation is enabled. | `string` | `true` | no |
| <a name="input_enabled"></a> [enabled](#input\_enabled) | Specifies whether the kms is enabled or disabled. | `bool` | `true` | no |
| <a name="input_environment"></a> [environment](#input\_environment) | Environment (e.g. `prod`, `dev`, `staging`). | `string` | `""` | no |
| <a name="input_is_enabled"></a> [is\_enabled](#input\_is\_enabled) | Specifies whether the key is enabled. | `bool` | `true` | no |
| <a name="input_key_material_base64"></a> [key\_material\_base64](#input\_key\_material\_base64) | Base64 encoded 256-bit symmetric encryption key material to import. The CMK is permanently associated with this key material. External key only | `string` | `null` | no |
| <a name="input_key_usage"></a> [key\_usage](#input\_key\_usage) | Specifies the intended use of the key. Defaults to ENCRYPT\_DECRYPT, and only symmetric encryption and decryption are supported. | `string` | `"ENCRYPT_DECRYPT"` | no |
| <a name="input_kms_key_enabled"></a> [kms\_key\_enabled](#input\_kms\_key\_enabled) | Specifies whether the kms is enabled or disabled. | `bool` | `true` | no |
| <a name="input_label_order"></a> [label\_order](#input\_label\_order) | label order, e.g. `name`,`application`. | `list(any)` | <pre>[<br>  "name",<br>  "environment"<br>]</pre> | no |
| <a name="input_managedby"></a> [managedby](#input\_managedby) | ManagedBy, eg 'slovink'. | `string` | `"slovink"` | no |
| <a name="input_multi_region"></a> [multi\_region](#input\_multi\_region) | Indicates whether the KMS key is a multi-Region (true) or regional (false) key. | `bool` | `true` | no |
| <a name="input_name"></a> [name](#input\_name) | Name  (e.g. `app` or `cluster`). | `string` | `""` | no |
| <a name="input_policy"></a> [policy](#input\_policy) | A valid policy JSON document. Although this is a key policy, not an IAM policy, an `aws_iam_policy_document`, in the form that designates a principal, can be used | `string` | `null` | no |
| <a name="input_primary_external_key_arn"></a> [primary\_external\_key\_arn](#input\_primary\_external\_key\_arn) | The primary external key arn of a multi-region replica external key | `string` | `null` | no |
| <a name="input_primary_key_arn"></a> [primary\_key\_arn](#input\_primary\_key\_arn) | The primary key arn of a multi-region replica key | `string` | `""` | no |
| <a name="input_repository"></a> [repository](#input\_repository) | Terraform current module repo | `string` | `"https://github.com/slovink/terraform-aws-kms.git"` | no |
| <a name="input_valid_to"></a> [valid\_to](#input\_valid\_to) | Time at which the imported key material expires. When the key material expires, AWS KMS deletes the key material and the CMK becomes unusable. If not specified, key material does not expire | `string` | `"2024-01-02T15:04:05Z"` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_alias_arn"></a> [alias\_arn](#output\_alias\_arn) | Alias ARN. |
| <a name="output_alias_name"></a> [alias\_name](#output\_alias\_name) | Alias name. |
| <a name="output_key_arn"></a> [key\_arn](#output\_key\_arn) | Key ARN. |
| <a name="output_key_id"></a> [key\_id](#output\_key\_id) | Key ID. |
| <a name="output_tags"></a> [tags](#output\_tags) | A mapping of tags to assign to the resource. |
| <a name="output_target_key_id"></a> [target\_key\_id](#output\_target\_key\_id) | Identifier for the key for which the alias is for, can be either an ARN or key\_id. |
<!-- END_TF_DOCS -->
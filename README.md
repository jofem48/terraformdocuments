# AWS Terraform module

Terraform module to provision base infrastructure on AWS for on-boarding new applications. Module automates the process of provisionig various network and governance modules including VPC, Subnets, Security Groups, cloudwatch Log groups and IAM role/polices on AWS.

## Module wrappers

Users of this Terraform module can create multiple similar resources by using [`for_each`](https://www.terraform.io/language/meta-arguments/for_each) meta-argument within `module` block which became available in Terraform 0.13

### Provisioning VPC in AWS

In order to provision the VPC we have defined locals block to create variables based on values passed in tfvars file while running the terraform code. Locals block is created to generate tag_name, public-subnet-list and private-subnet-list.

- `tag_name` is a created by appending `env_name`, `bu_n1`, `bu_n2` and `sflc` variables.
- `public-subnet-list` use [flatten terraform function](https://www.terraform.io/language/functions/flatten) to fetch list of public subnets and combine them into a single list containing tier and cidr in a sequence.
- `private-subnet-list` use [flatten terraform function](https://www.terraform.io/language/functions/flatten) to fetch list of private subnets and combine them into a single list containing tier and cidr in a sequence.  

```hcl
PASTE CODE SNIPPET HERE FROM LINE 22 - 47 HERE
```

Datasource [aws_availability_zones](https://registry.terraform.io/providers/hashicorp/aws/2.34.0/docs/data-sources/availability_zones) is used to fetch list of available availability zones which can be used within the resources to provision various network, governance and compute services.

```hcl
PASTE CODE SNIPPET HERE FROM LINE 49 - 51 HERE
```

Datasource [aws_region](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) is used to current regio where the code is currently running. This helps us use code in multiple regions without updating region at multiple locations within the code. 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 53 HERE
```

Resource [aws_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc) is used to provision Virtual Private Cloud in the cloud. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `create_vpc` variable to provision VPC. In case create_vpc is set to false, count will be set to 0 and it will not be provisioned. In case create_vpc is set to true, count will be set to 1 which will provision single VPC. 

Provisioned VPC will use `cidr`, `instance_tenancy`, `enable_dns_hostnames`, `enable_dns_support`, `enable_classiclink`, `enable_classiclink_dns_support` and `enable_ipv6`. Each VPC will have Name tag and few default tags based on application. 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 55 - 72 HERE
```

### Adding additional IPV4 CIDR Blocks to VPC

Resource [aws_vpc_ipv4_cidr_block_association](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_ipv4_cidr_block_association) is used to associate additional IPv4 CIDR blocks with a VPC. When a VPC is created, a primary IPv4 CIDR block for the VPC must be specified. The `aws_vpc_ipv4_cidr_block_association` resource allows further IPv4 CIDR blocks to be added to the VPC. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `associate_ipv4_cidr_block` variable and length of `secondary_ipv4_cidr_blocks` to associate additional CIDR block.  

Resource will use `vpc_id` and `cidr_block`. `cidr_block` uses [element](https://www.terraform.io/language/functions/element) function which helps find the required `secondary_ipv4_cidr_blocks` based on count index. 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 74 - 82 HERE
```

### Adding additional IPV6 CIDR Blocks to VPC

Resource [aws_vpc_ipv6_cidr_block_association](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_ipv6_cidr_block_association) is used to associate additional IPv6 CIDR blocks with a VPC. When a VPC is created, a primary IPv6 CIDR block for the VPC must be specified. The `aws_vpc_ipv6_cidr_block_association` resource allows further IPv6 CIDR blocks to be added to the VPC. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `associate_ipv6_cidr_block` variable and length of `secondary_ipv6_cidr_blocks` to associate additional CIDR block.  

Resource will use `vpc_id`, `ipv6_cidr_block` and `ipv6_ipam_pool_id`. `ipv6_cidr_block` uses [element](https://www.terraform.io/language/functions/element) function which helps find the required `secondary_ipv6_cidr_blocks` based on count index. `vpc_id` 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 74 - 82 HERE
```

### Create DHCP Options Set

Resource [aws_vpc_dhcp_options](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_dhcp_options) is used to provide VPC DHCP Options resource. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `enable_dhcp_options` variable. In case enable_dhcp_options is set to false, count will be set to 0 and it will not be provisioned. In case enable_dhcp_options is set to true, count will be set to 1 which will provision single DHCP options. 

Resource will use `n1fordns`, `domain_name_servers`, `ntp_servers`, `netbios_name_servers` and `netbios_node_type`. Each DHCP Options will have Name tag and few default tags based on application. 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 99 - 113 HERE
```

### Associate DHCP Options Set to VPC

Resource [aws_vpc_dhcp_options_association](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_dhcp_options_association) is used to associate VPC DHCP Options with VPC. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `associate_dhcp_options` variable. In case associate_dhcp_options is set to false, count will be set to 0 and it will not be provisioned. In case associate_dhcp_options is set to true, count will be set to 1 which will associate DHCP options with VPC. 

Resource will use `vpc_id`. `vpc_id` can fetch ID from end user or can automatically fetch the VPC ID provisioned in initial deployement. `dhcp_options_id` is automatically fetched from provisioned VPC resource.

```hcl
PASTE CODE SNIPPET HERE FROM LINE 120 - 126 HERE
```

### Create Public Subnet inside VPC

Resource [aws_subnet](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet) is used to provision Subnet within VPC. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `create_public_subnets` variable and length of `public-subnet-list`.  

Resource will use `vpc_id` and `map_public_ip_on_launch`. `cidr_block` and `availability_zone` are automatically pulled in via locals and variable declaration. Each subnet will have Name tag and few default tags based on application. 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 132 - 147 HERE
```

### Create Private Subnet inside VPC

Resource [aws_subnet](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet) is used to provision Subnet within VPC. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `create_private_subnets` variable and length of `private-subnet-list`.  

Resource will use `vpc_id` and `map_public_ip_on_launch`. `cidr_block` and `availability_zone` are automatically pulled in via locals and variable declaration. By default no public ip will be mapped to the provisioned resources. Each subnet will have Name tag and few default tags based on application. 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 153 - 166 HERE
```

### Create AWS Security Groups

Resource [aws_security_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) is used to provision a Security Group resource. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `create_security_group` variable. In case create_security_group is set to true, only then security group will be created else it will not be provisioned. We are adding Ingress and Egress rules via dynamic block which will help us loop over a map to add n number of rules without writting the code again and again.

Resource will use `vpc_id`, `aws_security_group_description`, `env_name`, `bu_n1`, `bu_n2`, `aws_security_group_name` , `security_group_ingress` and `security_group_egress`. Each security group will have Name tag and few default tags based on application. 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 173 - 217 HERE
```
### Creating VPC Flow Logs

Resource [aws_flow_log](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/flow_log) is used to provides a VPC/Subnet/ENI/Transit Gateway/Transit Gateway Attachment Flow Log to capture IP traffic for a specific network interface, subnet, or VPC. Logs are sent to a CloudWatch Log Group or a S3 Bucket. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `enable_flow_log` variable.  

Resource will use `vpc_id`. `iam_role_arn` fetch the arn from the IAM role which will be provisioned within same terraform configuration. `log_destination` will be fetched from the cloudwatch log group which will be provisioned within same terraform configuration. `traffic_type` is set to ALL. Each VPC Flow log will have few default tags based on application.

```hcl
PASTE CODE SNIPPET HERE FROM LINE 223 - 237 HERE
```

### Creating AWS Cloudwatch Log Group

Resource [aws_cloudwatch_log_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_group) is used to provision CloudWatch Log Group. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `enable_flow_log` variable.  

Resource will use `env_name`, `bu_n1`, `bu_n2` and `sdlc` variables. Each VPC log group will have few default tags based on application.

```hcl
PASTE CODE SNIPPET HERE FROM LINE 243 - 255 HERE
```

### Creating AWS Cloudwatch Log Stream

Resource [aws_cloudwatch_log_stream](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_stream) is used to provision CloudWatch Log Group. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `enable_flow_log` variable.  

Resource will use `env_name`, `bu_n1`, `bu_n2`, `log_group_name` and `sdlc` variables.

```hcl
PASTE CODE SNIPPET HERE FROM LINE 261 - 268 HERE
```

### Create IAM Role for VPC Flow Log

Resource [aws_iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/aws_iam_role) is used to create IAM role. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `enable_flow_log` variable.  

Resource will use `env_name`, `bu_n1`, `bu_n2` and `sdlc` variables. IAM role is created with the below mentioned default policy.

```hcl
PASTE CODE SNIPPET HERE FROM LINE 274 - 296 HERE
```

### Create IAM Role Policy for VPC Flow Log

Resource [aws_iam_role_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/aws_iam_role_policy) is used to create IAM role policy. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `enable_flow_log` variable.  

Resource will use `env_name`, `bu_n1`, `bu_n2` and `sdlc` variables. IAM role is created with the below mentioned default policy.

```hcl
PASTE CODE SNIPPET HERE FROM LINE 302 - 328 HERE
```

### Create AWS Cloudwatch Log Subscription Filter

Resource [aws_cloudwatch_log_subscription_filter](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_subscription_filter) is used to create CloudWatch Logs subscription filter. We are currently using a conditional [count](https://www.terraform.io/language/meta-arguments/count) block which depends the `enable_flow_log` variable.  

Resource will use `env_name`, `bu_n1`, `bu_n2`, `cwsubfilter_destination_arn` and `sdlc` variables. 

```hcl
PASTE CODE SNIPPET HERE FROM LINE 334 - 343 HERE
```

### Create AWS EC2 Managed Prefix List

Resource [aws_ec2_managed_prefix_list](https://registry.terraform.io/providers/hashicorp/aws/3.25.0/docs/resources/ec2_managed_prefix_list) is used to create a managed prefix list. We are currently using a conditional [for_each](https://www.terraform.io/language/meta-arguments/for_each) block which depends the `prefix_list` variable.  

Resource will use `max_entries`. `name` and `cidr` variable are fetched from the `prefix_list` map variable. Each EC2 Managed Prefix list will have few default tags based on application.

```hcl
PASTE CODE SNIPPET HERE FROM LINE 349 - 372 HERE
```

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.13.1 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 4.20.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 4.20.0 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [aws_availability_zones](https://registry.terraform.io/providers/hashicorp/aws/2.34.0/docs/data-sources/availability_zones) | data_source |
| [aws_region](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) | data_source |
| [aws_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc) | resource |
| [aws_vpc_ipv4_cidr_block_association](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_ipv4_cidr_block_association) | resource |
| [aws_vpc_ipv6_cidr_block_association](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_ipv6_cidr_block_association) | resource |
| [aws_vpc_dhcp_options](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_dhcp_options) | resource |
| [aws_vpc_dhcp_options_association](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_dhcp_options_association) | resource |
| [aws_subnet](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet) | resource |
| [aws_security_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) | resource |
| [aws_flow_log](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/flow_log) | resource |
| [aws_cloudwatch_log_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_group) | resource |
| [aws_cloudwatch_log_stream](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_stream) | resource |
| [aws_iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/aws_iam_role) | resource |
| [aws_iam_role_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/aws_iam_role_policy) | resource |
| [aws_cloudwatch_log_subscription_filter](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_subscription_filter) | resource |
| [aws_ec2_managed_prefix_list](https://registry.terraform.io/providers/hashicorp/aws/3.25.0/docs/resources/ec2_managed_prefix_list) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_ami"></a> [ami](#input\_ami) | ID of AMI to use for the instance | `string` | `""` | no |
| <a name="env_name"></a> [env_name](#input\_env_name) | 3-character name of environment (i.e. tec, ted) | `string` | `""` | yes |

## Outputs

| Name | Description |
|------|-------------|
| <a name="vpc_id"></a> [vpc_id](#output\_vpc_id) | The ID of the VCP |
| <a name="vpc_arn"></a> [vpc_arn](#output\_vpc_arn) | VPC ARN of the provisioned VPC |
| <a name="public_subnets"></a> [public_subnets](#output\_public_subnets) | List of IDs of public subnets |
| <a name="private_subnets"></a> [private_subnets](#output\_private_subnets) | List of IDs of private subnets |
| <a name="security_group_id"></a> [security_group_id](#output\_security_group_id) | VPC ID of the Security Group created |
<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Authors

Module is maintained by NAME-HERE.

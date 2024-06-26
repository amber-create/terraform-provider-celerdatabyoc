---
# generated by https://github.com/hashicorp/terraform-plugin-docs
page_title: "celerdatabyoc_classic_cluster Resource - terraform-provider-celerdatabyoc"
subcategory: "Deployment"
description: |-
  
---

~> The resource's API may change in subsequent versions to simplify user experience.

Deploys a classic CelerData cluster on AWS EC2 instances.

If this is your first cluster and it consists of a single FE node (instance type: m6i.xlarge) and a single BE node (instance type: m5.xlarge), this cluster will be automatically created as a [Free Developer Tier](https://docs.celerdata.com/en-us/main/get_started/free_developer_tier) cluster with which you can experience the features of CelerData Cloud Private at minimum costs.

This resource depends on the following resources and data source:

- [celerdatabyoc_aws_data_credential_policy](../resources/aws_data_credential_policy.md)
- [aws_iam_role (celerdata_data_cred_role)](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)
- [celerdatabyoc_aws_data_credential](../resources/aws_data_credential.md)
- [aws_iam_instance_profile](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_instance_profile)
- [celerdatabyoc_aws_deployment_credential_policy](../resources/aws_deployment_credential_policy.md)
- [aws_iam_role (deploy_cred_role)](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)
- [celerdatabyoc_aws_deployment_role_credential](../resources/aws_deployment_role_credential.md)
- [celerdatabyoc_aws_network](../resources/aws_network.md)

### Supported Instance Types

<html>
 <head></head>
 <body>
  <table>
   <tbody>
    <tr>
     <td rowspan="2"></td>
     <td rowspan="2">Instance type</td>
     <td rowspan="2">Instance size</td>
    </tr>
    <tr>
    </tr>
    <tr>
     <td rowspan="7">FE</td>
    </tr>
    <tr>
    </tr>
    <tr>
     <td>m6i.xlarge</td>
     <td>4 Cores 16GB Memory 50 GB Storage</td>
    </tr>
    <tr>
     <td>m6i.2xlarge</td>
     <td>8 Cores 32GB Memory 200 GB Storage</td>
    </tr>
    <tr>
     <td>m6i.4xlarge</td>
     <td>16 Cores 64GB Memory 50 GB Storage</td>
    </tr>
    <tr>
     <td>m6i.8xlarge</td>
     <td>32 Cores 128GB Memory 50 GB Storage</td>
    </tr>
    <tr>
    </tr>
    <tr>
     <td rowspan="6">BE (General Purpose)</td>
    </tr>
    <tr>
     <td>m5.xlarge</td>
     <td>4 Cores 16GB Memory</td>
    </tr>
    <tr>
     <td>m5.2xlarge</td>
     <td>8 Cores 32GB Memory</td>
    </tr>
    <tr>
     <td>m5.4xlarge</td>
     <td>16 Cores 64GB Memory</td>
    </tr>
    <tr>
     <td>m5.8xlarge</td>
     <td>32 Cores 128GB Memory</td>
    </tr>
    <tr>
    </tr>
    <tr>
     <td rowspan="7">BE (Memory Optimized)</td>
    <tr>
     <td>r6i.xlarge</td>
     <td>4 Cores 32GB Memory</td>
    </tr>
    <tr>
     <td>r6i.2xlarge</td>
     <td>8 Cores 64GB Memory</td>
    </tr>
    <tr>
     <td>r6i.4xlarge</td>
     <td>16 Cores 128GB Memory</td>
    </tr>
    <tr>
     <td>r6i.8xlarge</td>
     <td>32 Cores 256GB Memory</td>
    </tr>
   </tbody>
  </table>
 </body>
</html>

## Example Usage

```terraform
# Prerequisites for the celerdatabyoc_classic_cluster resource

data "celerdatabyoc_aws_data_credential_assume_policy" "assume_role" {}

resource "celerdatabyoc_aws_data_credential_policy" "role_policy" {
  bucket = local.s3_bucket
}

resource "aws_iam_role" "celerdata_data_cred_role" {
  name               = "<celerdata_data_credential_role_name>"
  assume_role_policy = data.celerdatabyoc_aws_data_credential_assume_policy.assume_role.json
  description        = "<celerdata_data_credential_role_description>"
  inline_policy {
    name   = "<celerdata_data_credential_role_policy_name>"
    policy = celerdatabyoc_aws_data_credential_policy.role_policy.json
  }
}

resource "celerdatabyoc_aws_data_credential" "data_credential" {
  name = "<celerdata_data_credential_name>"
  role_arn = aws_iam_role.celerdata_data_cred_role.arn
  instance_profile_arn = aws_iam_instance_profile.celerdata_data_cred_profile.arn
  bucket_name = local.s3_bucket
  policy_version = celerdatabyoc_aws_data_credential_policy.role_policy.version
}

resource "aws_iam_instance_profile" "celerdata_data_cred_profile" {
  name = "<celerdata_data_credential_profile_name>"
  role = aws_iam_role.celerdata_data_cred_role.name
}

resource "celerdatabyoc_aws_deployment_credential_policy" "role_policy" {
  bucket = local.s3_bucket
  data_role_arn = aws_iam_role.celerdata_data_cred_role.arn
}

resource "celerdatabyoc_aws_deployment_credential_assume_policy" "role_policy" {}

resource "aws_iam_role" "deploy_cred_role" {
  name               = "<celerdata_deployment_credential_role_name>"
  assume_role_policy = celerdatabyoc_aws_deployment_credential_assume_policy.role_policy.json
  description        = "<celerdata_deployment_credential_role_description>"
  inline_policy {
    name   = "<celerdata_deployment_credential_role_policy_name>"
    policy = celerdatabyoc_aws_deployment_credential_policy.role_policy.json
  }
}

resource "celerdatabyoc_aws_deployment_role_credential" "deployment_role_credential" {
  name = "<celerdata_deployment_credential_name>"
  role_arn = aws_iam_role.deploy_cred_role.arn
  external_id = celerdatabyoc_aws_deployment_credential_assume_policy.role_policy.external_id
  policy_version = celerdatabyoc_aws_deployment_credential_policy.role_policy.version
}

resource "celerdatabyoc_aws_network" "network" {
  name = "<VPC_name>"
  subnet_id = "<subnet_id>"
  security_group_id = "<security_group_id>"
  region = "<AWS_VPC_region>"
  deployment_credential_id = celerdatabyoc_aws_deployment_role_credential.deployment_role_credential.id
  vpc_endpoint_id = "<vpc_endpoint_id>"
}

# The celerdatabyoc_classic_cluster resource

resource "celerdatabyoc_classic_cluster" "classic_cluster_1" {
  cluster_name = "<cluster_name>"
  fe_instance_type = "<fe_node_instance_type>"
  fe_node_count = 1
  deployment_credential_id = celerdatabyoc_aws_deployment_role_credential.deployment_role_credential.id
  data_credential_id = celerdatabyoc_aws_data_credential.data_credential.id
  network_id = celerdatabyoc_aws_network.network.id
  be_instance_type = "<be_node_instance_type>"
  be_node_count = 1
  be_disk_number = 2
  be_disk_per_size = 100
  default_admin_password = "<SQL_user_initial_password>"

  expected_cluster_state = "{Suspended | Running}"
  resource_tags = {
    celerdata = "<tag_name>"
  }
  csp = "aws"
  region = "<AWS_VPC_region>"

  init_scripts {
      logs_dir    = "<log_s3_path>"
      script_path = "<script_s3_path>"
  }
  run_scripts_parallel = false
  query_port = 9030
  idle_suspend_interval = 64
}
```

## Argument Reference

### Data credential-related resources

#### celerdatabyoc_aws_data_credential_policy

This resource contains only the following required argument:

- `bucket`: (Forces new resource) The name of the AWS S3 bucket for which to generate the JSON policy document and that stores query profiles. Set the value to `local.s3_bucket`, as we recommend that you set the bucket element as a local value `s3_bucket` in your Terraform configuration. See [Local Values](https://developer.hashicorp.com/terraform/language/values/locals).

#### aws_iam_role (celerdata_data_cred_role)

This resource contains the following required arguments and optional arguments:

**Required:**

- `assume_role_policy`: The policy that grants an entity permission to assume the IAM role referenced in the data credential. Set the value to `data.celerdatabyoc_aws_data_credential_assume_policy.assume_role.json`.

**Optional:**

- `name`: (Forces new resource) The name of the IAM role referenced in the data credential. Enter a unique name. If omitted, Terraform will assign a random, unique name. See [IAM Identifiers](https://docs.aws.amazon.com/IAM/latest/UserGuide/Using_Identifiers.html) for more information.
- `description`: The description of the IAM role.
- `inline_policy`: The configuration block that defines an exclusive set of IAM inline policies associated with the IAM role. See below. If no blocks are configured, Terraform will not manage any inline policies in this resource. Configuring one empty block (namely, `inline_policy {}`) will cause Terraform to remove all inline policies added out of band on `apply`.
  - `name`: The name of the IAM policy that will be attached to the IAM role referenced in the data credential.
  - `policy`: The IAM policy that will be attached to the IAM role. Set the value to `celerdatabyoc_aws_data_credential_policy.role_policy.json`.

#### celerdatabyoc_aws_data_credential

This resource contains the following required arguments and optional arguments:

**Required:**

- `role_arn`: (Forces new resource) The ARN of the IAM role referenced in the data credential. Set the value to `aws_iam_role.celerdata_data_cred_role.arn`.
- `instance_profile_arn`: (Forces new resource) The instance profile ARN of the IAM role referenced in the data credential. Set the value to `aws_iam_instance_profile.celerdata_data_cred_profile.arn`.
- `bucket_name`: (Forces new resource) The name of the AWS S3 bucket for which to generate the policy document and that stores query profiles. Set the value to `local.s3_bucket`, as we recommend that you set the bucket element as a local value `s3_bucket` in your Terraform configuration. See [Local Values](https://developer.hashicorp.com/terraform/language/values/locals).
- `policy_version`: (Forces new resource) Set the value to `celerdatabyoc_aws_data_credential_policy.role_policy.version`.

**Optional:**

- `name`: (Forces new resource) The name of the data credential. Enter a unique name. If omitted, Terraform will assign a random, unique name.

### Deployment credential-related resources

#### aws_iam_instance_profile

This resource contains only the following optional arguments:

- `name`: (Forces new resource) The name of the instance profile. Enter a unique name. If omitted, Terraform will assign a random, unique name. This argument conflicts with `name_prefix`. The value of this argument can be a string of characters consisting of upper and lowercase alphanumeric characters and these special characters: `_`, `+`, `=`, `,`, `.`, `@`, `-`. Spaces are not allowed.
- `role`: The name of the IAM role to add to the instance profile. Set the value to `aws_iam_role.celerdata_data_cred_role.name`.

#### celerdatabyoc_aws_deployment_credential_policy

This resource contains only the following required arguments:

- `bucket`: The name of the AWS S3 bucket. Set the value to `local.s3_bucket`, as we recommend that you set the bucket element as a local value `s3_bucket` in your Terraform configuration. See [Local Values](https://developer.hashicorp.com/terraform/language/values/locals).
- `data_role_arn`: (Forces new resource) The ARN of the IAM role referenced in the deployment credential. Set the value to `aws_iam_role.celerdata_data_cred_role.arn`.

#### aws_iam_role (deploy_cred_role)

This resource contains the following required arguments and optional arguments:

**Required:**

- `assume_role_policy`: The policy that grants an entity permission to assume the IAM role referenced in the deployment credential. Set the value to `celerdatabyoc_aws_deployment_credential_assume_policy.role_policy.json`.

**Optional:**

- `name`: (Forces new resource) The name of the IAM role referenced in the deployment credential. Enter a unique name. If omitted, Terraform will assign a random, unique name. See [IAM Identifiers](https://docs.aws.amazon.com/IAM/latest/UserGuide/Using_Identifiers.html) for more information.
- `description`: The description of the IAM role.
- `inline_policy`: The configuration block that defines an exclusive set of IAM inline policies associated with the IAM role. See below. If no blocks are configured, Terraform will not manage any inline policies in this resource. Configuring one empty block (namely, `inline_policy {}`) will cause Terraform to remove all inline policies added out of band on `apply`.
  - `name`: The name of the IAM policy that will be attached to the IAM role.
  - `policy`: The IAM policy that will be attached to the IAM role referenced in the deployment credential. Set the value to `celerdatabyoc_aws_deployment_credential_policy.role_policy.json`.

#### celerdatabyoc_aws_deployment_role_credential

This resource contains the following required arguments and optional arguments:

**Required:**

- `role_arn`: (Forces new resource) Set the value to `aws_iam_role.deploy_cred_role.arn`.
- `external_id`: (Forces new resource) Set the value to `celerdatabyoc_aws_deployment_credential_assume_policy.role_policy.external_id`.
- `policy_version`: (Forces new resource) Set the value to `celerdatabyoc_aws_deployment_credential_policy.role_policy.version`.

**Optional:**

- `name`: (Forces new resource) The name of the deployment credential. Enter a unique name. If omitted, Terraform will assign a random, unique name.

### Network configuration-related resources

The `celerdatabyoc_aws_network` resource contains the following required arguments and optional arguments:

**Required:**

- `name`: (Forces new resource) The name of the AWS VPC hosting the cluster. Enter a unique name.

- `subnet_id`: (Forces new resource) The ID of the subnet within the AWS VPC.

- `security_group_id`: (Forces new resource) The ID of the security group within the AWS VPC.

- `region`: (Forces new resource) The ID of the cloud provider region to which the network hosting the cluster belongs. See [Supported cloud platforms and regions](https://docs.celerdata.com/private/main/get_started/cloud_platforms_and_regions).

- `deployment_credential_id`: (Forces new resource) Set the value to `celerdatabyoc_aws_deployment_role_credential.deployment_role_credential.id`.

**Optional:**

- `vpc_endpoint_id`: (Optional) The ID of your endpoint within your VPC. Set this argument if you need to achieve a more stringent network communication method.

### CelerData cluster-related resources

The `celerdatabyoc_classic_cluster` resource contains the following required arguments and optional arguments:

**Required:**

- `cluster_name`: (Forces new resource) The desired name for the cluster. Enter a unique name.

- `fe_instance_type`: The instance type for FE nodes in the cluster. Select an FE instance type from the table "[Supported Instance Types](#supported-instance-types)". For example, you can set this argument to `m6i.4xlarge`.

- `deployment_credential_id`: (Forces new resource) Set the value to `celerdatabyoc_aws_deployment_role_credential.deployment_role_credential.id`.

- `data_credential_id`: (Forces new resource) Set the value to `celerdatabyoc_aws_data_credential.data_credential.id`.

- `network_id`: (Forces new resource) Set the value to `celerdatabyoc_aws_network.network.id`.

- `be_instance_type`: The instance type for BE nodes in the cluster. Select a BE instance type from the table "[Supported Instance Types](#supported-instance-types)". For example, you can set this argument to `m5.xlarge`.

- `default_admin_password`: The initial password of the cluster `admin` user.

- `expected_cluster_state`: When creating a cluster, you need to declare the status of the cluster you are creating. Cluster states are categorized as `Suspended` and `Running`. If you want the cluster to start after provisioning, set this argument to `Running`. If you do not do so, the cluster will be suspended after provisioning.

- `csp`: The cloud service provider of the cluster. Only AWS is supported.

- `region`: The ID of the AWS region to which the AWS VPC hosting the cluster belongs. The following AWS regions are supported.

  | Region                   | Region ID      |
  | ------------------------ | -------------- |
  | Asia Pacific (Singapore) | ap-southeast-1 |
  | US East (N. Virginia)    | us-east-1      |
  | US West (Oregon)         | us-west-2      |
  | Europe (Ireland)         | eu-west-1      |
  | Europe (Frankfurt)       | eu-central-1   |

**Optional:**

- `fe_node_count`: The number of FE nodes in the cluster. Valid values: `1`, `3`, and `5`. Default value: `1`.
- `be_node_count`: The number of BE nodes in the cluster. Valid values: any non-zero positive integer. Default value: `3`.
- `be_disk_number`: (Forces new resource) The maximum number of disks that are allowed for each BE. Valid values: [1,24]. Default value: `2`.
- `be_disk_per_size`: The size per disk for each BE. Unit: GB. Maximum value: `16000`. Default value: `100`. You can only increase the value of this parameter, and the time interval between two value changes must be greater than 6 hours.
- `resource_tags`: The tags to be attached to the cluster.
- `init_scripts`: The configuration block to specify the paths to which scripts and script execution results are stored. The maximum number of executable scripts is 20. For information about the formats supported by these arguments, see `scripts.logs_dir` and `scripts.script_path` in [Run scripts](https://docs.celerdata.com/en-us/main/run_scripts).
  - `logs_dir`: (Forces new resource) The path in the AWS S3 bucket to which script execution results are stored. This S3 bucket can be the same as or different from the S3 bucket you specify in the `celerdatabyoc_aws_data_credential` resource.
  - `script_path`: (Forces new resource) The path in the AWS S3 bucket that stores the scripts to run via Terraform. This S3 bucket must be the one you specify in the `celerdatabyoc_aws_data_credential` resource.
- `run_scripts_parallel`: Whether to execute the scripts in parallel. Valid values: `true` and `false`. Default value: `false`.
- `query_port`: The query port, which must be within the range of 1-65535 excluding 443. The default query port is port 9030. Note that this argument can be specified only at cluster deployment, and cannot be modified once it is set.
- `idle_suspend_interval`: The amount of time (in minutes) during which the cluster can stay idle. After the specified time period elapses, the cluster will be automatically suspended. The Auto Suspend feature is disabled by default. To enable the Auto Suspend feature, set this argument to an integer with the range of 60-999999. To disable this feature again, remove this argument from your Terraform configuration.

## See Also

- [AWS IAM](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/policies)
- [Manage data credentials for AWS](https://docs.celerdata.com/en-us/main/cloud_settings/aws_cloud_settings/manage_aws_data_credentials)
- [Manage deployment credentials for AWS](https://docs.celerdata.com/en-us/main/cloud_settings/aws_cloud_settings/manage_aws_deployment_credentials)
- [Manage network configurations for AWS](https://docs.celerdata.com/en-us/main/cloud_settings/aws_cloud_settings/manage_aws_network_configurations)

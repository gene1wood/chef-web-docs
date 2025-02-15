+++
title = "AWS Deployment"

draft = false

gh_repo = "automate"

[menu]
  [menu.automate]
    title = "AWS Deployment"
    parent = "automate/deploy_high_availability/deployment"
    identifier = "automate/deploy_high_availability/deployment/ha_aws_deploy_steps.md AWS Deployment"
    weight = 210
+++

{{< warning >}}
{{% automate/ha-warn %}}
{{< /warning >}}

Follow the steps below to deploy Chef Automate High Availability (HA) on AWS (Amazon Web Services) cloud.

## Install Chef Automate HA on AWS

### Prerequisites

- Virtual Private Cloud (VPC) should be created in AWS before starting or use default. Reference for [VPC and CIDR creation](/automate/ha_vpc_setup/)
- Get AWS credentials (`aws_access_key_id` and `aws_secret_access_key`) which have privileges like: `AmazonS3FullAccess`, `AdministratorAccess`, `AmazonAPIGatewayAdministrator`. \
    Set these in `~/.aws/credentials` in Bastion Host:

    ```bash
    sudo su -
    ```

    ```bash
    mkdir -p ~/.aws
    echo "aws_access_key_id=<ACCESS_KEY_ID>" >> ~/.aws/credentials
    echo "aws_secret_access_key=<SECRET_KEY>" >> ~/.aws/credentials
    ```

- Have DNS certificate ready in ACM for 2 DNS entries: Example: `chefautomate.example.com`, `chefinfraserver.example.com`
    Reference for [Creating new DNS Certificate in ACM](/automate/ha_aws_cert_mngr/)
- Have SSH Key Pair ready in AWS, so new VM's are created using that pair.
    Reference for [AWS SSH Key Pair creation](https://docs.aws.amazon.com/ground-station/latest/ug/create-ec2-ssh-key-pair.html)
- We do not support passphrase for Private Key authentication.

{{< warning >}} PLEASE DONOT MODIFY THE WORKSPACE PATH it should always be "/hab/a2_deploy_workspace"
{{< /warning >}}

### Deployment

Run the following steps on Bastion Host Machine:

1. Run below commands to download latest Automate CLI and Airgapped Bundle:

   ```bash
   #Run commands as sudo.
   sudo -- sh -c "
   #Download Chef Automate CLI.
   curl https://packages.chef.io/files/current/latest/chef-automate-cli/chef-automate_linux_amd64.zip \
   | gunzip - > chef-automate && chmod +x chef-automate \
   | cp -f chef-automate /usr/bin/chef-automate

   #Download latest Airgapped Bundle.
   #To download specific version bundle, example version: 4.2.59 then replace latest.aib with 4.2.59.aib
   curl https://packages.chef.io/airgap_bundle/current/automate/latest.aib -o automate.aib

   #Generate init config and then generate init config for existing infra structure
   chef-automate init-config-ha aws
   "
   ```

   {{< note >}} Chef Automate bundles are available for 365 days from the release of a version. However, the milestone release bundles are available for download forever. {{< /note >}}

1. Update Config with relevant data. Click [here](/automate/ha_aws_deploy_steps/#sample-config) for sample config

   ```bash
   vi config.toml
   ```

   - Give `ssh_user` which has access to all the machines. Example: `ubuntu`
   - Give `ssh_port` in case your AMI is running on custom ssh port, default will be 22.
   - Give `ssh_key_file` path, this should have been download from AWS SSH Key Pair which we want to use to create all the VM's. Thus, we will be able to access all VM's using this.
   - `sudo_password` is only meant to switch to sudo user. If you have configured password for sudo user, please provide it here.
   - We support only private key authentication.
   - Set `backup_config` to `"efs"` or `"s3"`
   - If `backup_config` is `s3` then set `s3_bucketName` to a Unique Value.
   - Set `admin_password` which you can use to access Chef Automate UI for user `admin`.
   - Don't set `fqdn` for this AWS deployment.
   - Set `instance_count` for Chef Automate, Chef Infra Server, Postgresql, OpenSearch.
   - Set AWS Config Details:
     - Set `profile`, by default `profile` is `"default"`
     - Set `region`, by default `region` is `"us-east-1"`
     - Set `aws_vpc_id`, which you had created as Prerequisite step. Example: `"vpc12318h"`
     - If AWS VPC uses CIDR then set `aws_cidr_block_addr`.
     - If AWS VPC uses Subnet then set `private_custom_subnets` and `public_custom_subnets` Example: example : `["subnet-07e469d218301533","subnet-07e469d218041534","subnet-07e469d283041535"]`
     - Set `ssh_key_pair_name`, this is the SSH Key Pair we created as Prerequisite. This value should be just name of the AWS SSH Key Pair, not having `.pem` extention. The ssh key content should be same as content of `ssh_key_file`.
     - Set `setup_managed_services` as `false`, As these deployment steps are for Non-Managed Services AWS Deployment. Default value is `false`.
     - Set `ami_id`, this value depends on your AWS Region and the Operating System Image you want to use.
     - Please use the [Hardware Requirement Calculator sheet](/calculator/automate_ha_hardware_calculator.xlsx) to get information for which instance type you will need for your load.
     - Set Instance Type for Chef Automate in `automate_server_instance_type`.
     - Set Instance Type for Chef Infra Server in `chef_server_instance_type`.
     - Set Instance Type for OpenSearch in `opensearch_server_instance_type`.
     - Set Instance Type for Postgresql in `postgresql_server_instance_type`.
     - Set `automate_lb_certificate_arn` with the arn value of the Certificate created in AWS ACM for DNS entry of `chefautomate.example.com`.
     - Set `chef_server_lb_certificate_arn` with the arn value of the Certificate created in AWS ACM for DNS entry of `chefinfraserver.example.com`.
     - Set `automate_ebs_volume_iops`, `automate_ebs_volume_size` based on your load needs.
     - Set `chef_ebs_volume_iops`, `chef_ebs_volume_size` based on your load needs.
     - Set `opensearch_ebs_volume_iops`, `opensearch_ebs_volume_size` based on your load needs.
     - Set `postgresql_ebs_volume_iops`, `postgresql_ebs_volume_size` based on your load needs.
     - Set `automate_ebs_volume_type`, `chef_ebs_volume_type`, `opensearch_ebs_volume_type`, `postgresql_ebs_volume_type`. Default value is `"gp3"`. Change this based on your needs.

   {{< note >}} Click [here](/automate/ha_cert_deployment) to know more on adding certificates for services during deployment. {{< /note >}}

1. Continue with the deployment after updating config:

   ```bash
   #Run commands as sudo.
   sudo -- sh -c "
   #Print data in the config
   cat config.toml

   #Run provision command to deploy `automate.aib` with set `config.toml`
   chef-automate provision-infra config.toml --airgap-bundle automate.aib

   #Run deploy command to deploy `automate.aib` with set `config.toml`
   chef-automate deploy config.toml --airgap-bundle automate.aib

   #After Deployment is done successfully. Check status of Chef Automate HA services
   chef-automate status

   #Check Chef Automate HA deployment information, using the following command
   chef-automate info
      "
   ```

Note: DNS should have entry for `chefautomate.example.com` and `chefinfraserver.example.com` pointing to respective Load Balancers as shown in `chef-automate info` command.

Check if Chef Automate UI is accessible by going to (Domain used for Chef Automate) [https://chefautomate.example.com](https://chefautomate.example.com).

### Destroy infra

{{< danger >}}
Below section will destroy the infrastructure
{{< /danger >}}

#### To destroy AWS infra created with S3 Bucket

To destroy infra after successfull provisioning, run below command in your bastion host in same order.

1. This command will initialize the terraform packages

    ```bash
    for i in 1;do i=$PWD;cd /hab/a2_deploy_workspace/terraform/destroy/aws/;terraform init;cd $i;done
    ```

2. This command will destroy all resources created while provisioning (excluding S3).

    ```bash
    for i in 1;do i=$PWD;cd /hab/a2_deploy_workspace/terraform/destroy/aws/;terraform destroy;cd $i;done
    ```

#### To destroy AWS infra created with EFS Bucket

To destroy infra after successfull provisioning, run below command in your bastion host in same order.

1. This command will initialize the terraform packages

    ```bash
    for i in 1;do i=$PWD;cd /hab/a2_deploy_workspace/terraform/destroy/aws/;terraform init;cd $i;done
    ```

2. Following command will remove EFS from terraform state file, so that `destroy` command will not destroy EFS.

    ```bash
    for i in 1;do i=$PWD;cd /hab/a2_deploy_workspace/terraform/destroy/aws/;terraform state rm "module.efs[0].aws_efs_file_system.backups";cd $i;done
    ```

3. This command will destroy all resources created while provisioning (excluding EFS).

    ```bash
    for i in 1;do i=$PWD;cd /hab/a2_deploy_workspace/terraform/destroy/aws/;terraform destroy;cd $i;done
    ```

#### Sample config

{{< note >}}

- Assuming 8+1 nodes (1 bastion, 1 for automate UI, 1 for Chef-server, 3 for Postgresql, 3 for Opensearch)

{{< /note >}}

{{< note >}}

- User only needs to create/setup **the bastion node** with IAM role of Admin access, and s3 bucket access attached to it.
- It is adviceble to create bastion server (EC2 instance) in a new VPC.
- Following config will create s3 bucket for backup.

{{< /note >}}

```config
[architecture.aws]
ssh_port = "22"
ssh_user = ""
# Private SSH key file path, which has access to all the instances.
# Eg.: ssh_key_file = "~/.ssh/A2HA.pem"
ssh_key_file = ""
# Eg.: backup_config = "efs" or "s3"
backup_config = "s3"
secrets_key_file = "/hab/a2_deploy_workspace/secrets.key"
secrets_store_file = "/hab/a2_deploy_workspace/secrets.json"
architecture = "aws"
# DON'T MODIFY THE BELOW LINE (workspace_path)
workspace_path = "/hab/a2_deploy_workspace"
backup_mount = "/mnt/automate_backups"
[automate.config]
admin_password = ""
# Automate Load Balancer FQDN eg.: "chefautomate.example.com"
fqdn = ""
instance_count = "1"
config_file = "configs/automate.toml"
# Set enable_custom_certs = true to provide custom certificates during deployment
enable_custom_certs = false
# Add Automate load balancer root-ca and keys
# root_ca = ""
# private_key = ""
# public_key = ""
[chef_server.config]
instance_count = "1"
# Set enable_custom_certs = true to provide custom certificates during deployment
enable_custom_certs = false
# Add Chef Server load balancer root-ca and keys
# root_ca = ""
# private_key = ""
# public_key = ""
[opensearch.config]
instance_count = "3"
# Set enable_custom_certs = true to provide custom certificates during deployment
enable_custom_certs = false
# Add OpenSearch load balancer root-ca and keys
# root_ca = ""
# admin_key = ""
# admin_cert = ""
# private_key = ""
# public_key = ""
[postgresql.config]
instance_count = "3"
# Set enable_custom_certs = true to provide custom certificates during deployment
enable_custom_certs = false
# Add Postgresql load balancer root-ca and keys
# root_ca = ""
# private_key = ""
# public_key = ""
[aws.config]
profile = "default"
# Eg.: region = "us-east-1"
region = ""
aws_vpc_id  = ""
aws_cidr_block_addr  = ""
private_custom_subnets = []
public_custom_subnets = []
# ssh key pair name in AWS to access instances
# eg: ssh_key_pair_name = "A2HA"
ssh_key_pair_name = ""
# ============== EC2 Instance Config ===================
## === INPUT NEEDED ===
# This AMI should be from the Same Region which we selected above.
# eg: ami_id = "ami-08d4ac5b634553e16" # This ami is of Ubuntu 20.04 in us-east-1
ami_id = ""
automate_server_instance_type = "t3.medium"
chef_server_instance_type = "t3.medium"
opensearch_server_instance_type = "m5.large"
postgresql_server_instance_type = "t3.medium"
automate_lb_certificate_arn = ""
chef_server_lb_certificate_arn = ""
automate_ebs_volume_iops = "100"
automate_ebs_volume_size = "50"
automate_ebs_volume_type = "gp3"
chef_ebs_volume_iops = "100"
chef_ebs_volume_size = "50"
chef_ebs_volume_type = "gp3"
opensearch_ebs_volume_iops = "100"
opensearch_ebs_volume_size = "50"
opensearch_ebs_volume_type = "gp3"
postgresql_ebs_volume_iops = "100"
postgresql_ebs_volume_size = "50"
postgresql_ebs_volume_type = "gp3"
lb_access_logs = "false"
# ======================================================
# ============== EC2 Instance Tags =====================
X-Contact = ""
X-Dept = ""
X-Project = "Test_Project"
# ======================================================
```

##### Changes to be made

- Give `ssh_user` which has access to all the machines. Eg: `ubuntu`, `centos`, `ec2-user`
- Give `ssh_key_file` path, this key should have access to all the Machines or VM's. Eg: `~/.ssh/id_rsa`, `/home/ubuntu/key.pem`
- Give `fqdn` as the DNS entry of Chef Automate, which LoadBalancer redirects to Chef Automate Machines or VM's. (optional for above configuration) Eg: `chefautomate.example.com`
- Provide `region` Eg: `us-east-1`, `ap-northeast-1`.
- Provide `aws_vpc_id` Eg: `vpc-0a12*****`
- Provide `aws_cidr_block_addr` Eg: `10.0.192.0`
- Provide `ssh_key_pair_name` Eg: `user-key`
- Provide `ami_filter_name` Eg: `ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*`
- Provide `ami_filter_virt_type` Eg: `hvm`
- Provide `ami_filter_owner` Eg: `112758395563`
- Give `ami_id` for the respective region where the infra is been created. Eg: `ami-0bb66b6ba59664870`
- Provide `certificate ARN` for both automate and Chef server in `automate_lb_certificate_arn` and `chef_server_lb_certificate_arn` respectively.

### How to Add more nodes In AWS Deployment, post deployment.
The commands require some arguments so that it can determine which types of nodes you want to add to your HA setup from your bastion host. It needs the count of the nodes you want to add as as argument when you run the command.
For example,

- if you want to add 2 nodes to automate, you have to run the:

    ```sh
    chef-automate node add --automate-count 2
    ```

- If you want to add 3 nodes to chef-server, you have to run the:

    ```sh
    chef-automate node add --chef-server-count 3
    ```

- If you want to add 1 node to OpenSearch, you have to run the:

    ```sh
    chef-automate node add --opensearch-count 1
    ```

- If you want to add 2 nodes to PostgreSQL you have to run:

    ```sh
    chef-automate node add --postgresql-count 2
    ```

You can mix and match different services if you want to add nodes across various services.

- If you want to add 1 node to automate and 2 nodes to PostgreSQL, you have to run:

    ```sh
    chef-automate node add --automate-count 1 --postgresql-count 2
    ```

- If you want to add 1 node to automate, 2 nodes to chef-server, and 2 nodes to PostgreSQL you have to run:

    ```sh
    chef-automate node add --automate-count 1 --chef-server-count 2 --postgresql-count 2
    ```

Once the command executes, it will add the supplied number of nodes to your automate setup. The changes might take a while.

{{< note >}}

- If you have patched some external config to any of the existing services then make sure you apply the same on the new nodes as well.
For example, if you have patched any external configurations like SAML or LDAP, or any other done manually post-deployment in automate nodes, make sure to patch those configurations on the new automate nodes. The same must be followed for services like Chef-Server, Postgresql, and OpenSearch.
- The new node will be configured with the certificates which were already configured in your HA setup.

{{< /note >}}


{{< warning >}}
  Downgrade the number of instance_count for backend node will be data loss. We can not downgrade the backend node. 
{{< /warning >}}

### Uninstall chef automate HA

{{< danger >}}
- Running clean up command will remove all AWS resources created by `provision-infra` command
- Adding `--force` flag will remove storage (Object Storage/ NFS) if it is created by`provision-infra`
{{< /danger >}}

To uninstall chef automate HA instances after unsuccessfull deployment, run below command in your bastion host.
```bash
    chef-automate cleanup --aws-deployment --force
```
or
```bash
    chef-automate cleanup --aws-deployment
```

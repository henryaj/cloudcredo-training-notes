# Deploying Micro BOSH and CF to AWS

## Use `bosh aws` to provision infrastructure
* follow step ['Prepare the Deployment Environment'](https://docs.cloudfoundry.org/deploying/ec2/bootstrap-aws-vpc.html#deployment-env-prep) to export the required environment variables
* note that `BOSH_VPC_DOMAIN` and `BOSH_VPC_SUBDOMAIN` must be set, even when you do not have a domain, as these values are used to name the S3 buckets (which must be unique)
* run `bosh aws create` to provision the required infrastructure

## Deploying Micro BOSH
(Based on http://bosh.io/docs/deploy-microbosh-to-aws.html)

### Customise manifest
* using the example manifest.yml from the above link, customise the variables based on the values from the newly-created AWS infrastructure.
* ELASTIC-IP-ADDRESS
    - choose an unassociated elastic IP address to use
* SUBNET-ID
    - replace with the ID of the 'bosh1' subnet
* AVAILABILITY-ZONE
    - replace with the same AZ that the bosh1 subnet is running on
* ACCESS-KEY-ID and SECRET-ACCESS-KEY
    - you know what to do
* `ec2_private_key`
    - `bosh aws create` creates `~/.ssh/id_rsa_bosh`, so either that or copy it to `./bosh.pem`
* change all IP addresses from 10.0 to 10.10

### Download stemcell and deploy
* visit http://bosh.io/stemcells and pick one - pick a 'Light' stemcell (which just contains a reference to an AMI to be downloaded later). Also, HVM stemcells won't work because (OHGODWESTILLDON'TKNOW!).
* `bosh micro deployment <manifest.yml>`, then `bosh micro deploy <path_to_stemcell.tgz>`
* when the deployment is complete, target it using `bosh target <elastic_ip>`

### Deploy CF

## Do not create a manifest stub, do not pass GO, go straight to [Configure AWS to Cloud Foundry](https://docs.cloudfoundry.org/deploying/ec2/configure_aws_cf.html)

### Configure AWS for Your Cloud Foundry Deployment

These instructions override those given to line up the manifest with what `bosh aws create` sets up.

#### Create a NAT VM
* Create a NAT VM
* Configure a NAT security group
* Disable the Source/Dest. Check for the NAT VM

#### Update the BOSH Security Group
* No need to add the custom TCP Rule for Port 4443 to the `bosh` security group

#### Create a Subnet for Cloud Foundry Deployment
* No need to create a 'cf' subnet, use `cf1`
* Select the `cf1` subnet and edit the route table for `0.0.0.0/0` to route to the NAT instance
* No need to create a 'cf-public' security group, use `web`

#### Configure your Cloud Foundry System Domain
* Skip steps 1-5 of 'Configure your Cloud Foundry System Domain'
* Create an SSL certificate

### Configure `minimal-aws.yml`

* Replace all IP address starting `10.0.` with `10.10.` to match the subnet created by `bosh aws create`

* Replace the contents of `jobs:uaa_z1:properties:uaadb` with the values from `aws_rds_bosh_receipt.yml` under `deployment_manifest:properties:uaadb`

* Replace the contents of `properties:ccdb` with the values from `aws_rds_bosh_receipt.yml` under `deployment_manifest:properties:ccdb`

* Change the name and version of `resource_pools:small_z1:stemcell` to match the stemcell uploaded to Micro BOSH.

* REPLACE_WITH_DIRECTOR_ID
    - run `bosh status --uuid`

* REPLACE_WITH_BOSH_SECURITY_GROUP
    - id of the `bosh` security group (MAYBE it should be the name, i.e. `bosh`?)

* REPLACE_WITH_PRIVATE_SUBNET_ID
    - id of `cf1` subnet

* REPLACE_WITH_PUBLIC_SUBNET_ID
    - id of the `web` subnet

* REPLACE_WITH_AZ
    - availability zone of the `microbosh` instance

* REPLACE_WITH_ELASTIC_IP
    - choose one of the three unassociated elastic IPs (**not** the same one used for Micro BOSH!)

* REPLACE_WITH_PUBLIC_SECURITY_GROUP
    - id of the `web` security group (MAYBE it should be the name, i.e. `web`?)

* REPLACE_WITH_SYSTEM_DOMAIN
    - append `.xip.io` to the same unassociated elastic IP used for REPLACE_WITH_ELASTIC_IP

* REPLACE_WITH_SSL_CERT_AND_KEY
    - run the commands to generate an SSL certificate using REPLACE_WITH_SYSTEM_DOMAIN when prompted

### Deploy CF to Micro BOSH
* *(optional)* create a custom release, if desired
* upload the CF release to BOSH:
  `bosh upload release <path_to_cf_release.yml>`
* confirm the upload was successful: `bosh releases`
* using the deployment manifest you just customised:
  `bosh deployment <path_to_minimal_aws_manifest.yml>`
* deploy it! `bosh deploy`
* THEN IT WILL WORK


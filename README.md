# Provision Kubernetes Cluster With Terraform

In this workshop we develop Terraform script to setup Kubernetes Cluster.

## Steps
1) Collect the details of cloud attributes
2) Download the terraform script and extract it
3) Check the integrity of the code 
4) Execute the terraform code using Apply command

## 1) Collect the details of cloud attributes

#### Tenancy OCID 
Open the Profile menu (User menu icon)  and click Tenancy: <your_tenancy_name>.
The tenancy OCID is shown under Tenancy Information. Click Copy to copy it to your clipboard.

![Tenancy](https://github.com/kmkittu/OKE/blob/main/Tenancy.png)

The tenancy OCID looks something like this
ocid1.tenancy.oc1..<unique_ID>     (notice the word "tenancy" in it)

#### User OCID    
If you're signed in as the user to OCI console: Open the Profile menu (User menu icon)   and click User Settings.
If you're an administrator doing this for another user: Open the navigation menu. Under Governance and Administration, go to Identity and click Users. Select the user from the list.
The user OCID is shown under User Information. Click Copy to copy it to your clipboard.

![User OCID](https://github.com/kmkittu/OKE/blob/main/User%20OCID.png)

#### Private key 

SSH key pair is required to login into OCI console
If SSH key pair is not created, then follow the below steps.
Login into any Linux machine and execute openssh command.
    $ openssh genrsa -out oci_key.pem 2048 
    Generating RSA private key, 2048 bit long modulus
    ...................................................................................+++
    .....+++
    e is 65537 (0x10001)

-out denotes output location of the generated private key.  In the above example, oci_key.pem is the private key. 

    [oracle@db key]$ ls -lrt
    -rw-r--r-- 1 oracle oinstall 1679 Apr  3 07:35 oci_key.pem

Generate Public Key with Pem(Privacy Enhanced Mail) format

    [oracle@db key]$ openssh rsa -pubout -in oci_key.pem -out oci_key_public.pem
    writing RSA key
    [oracle@db key]$ ls -lrt
    -rw-r--r-- 1 oracle oinstall 1679 Apr  3 07:35 oci_key.pem
    -rw-r--r-- 1 oracle oinstall  451 Apr  3 07:40 oci_key_public.pem

Login into OCI cloud consile, click user settings. 

![user settings ] (https://github.com/kmkittu/OKE/blob/main/user%20settings.png)

In the page click API Key

![api key] (https://github.com/kmkittu/OKE/blob/main/user%20settings.png)

Click "Add API Key" button

![api key button](https://github.com/kmkittu/OKE/blob/main/user%20add%20public%20key.png)

Now our public key become part of OCI. Once the key is added it will listed along with Fingerprint.

#### Fingerprint   
You can get the key's fingerprint with the following OpenSSL command. If you're using Windows, you'll need to install Git Bash for Windows and run the command with that tool.

    openssl rsa -pubout -outform DER -in oci_key_public.pem | openssl md5 -c

Also in other way when you upload the public key in the Console (As you did in the user details page) , the fingerprint is also automatically displayed there. It looks something like this: 12:34:56:78:90:ab:cd:ef:12:34:56:78:90:ab:cd:ef

![Fingerprint](https://github.com/kmkittu/OKE/blob/main/Add%20public%20Key%20-%20Fingerprint.png)

### SSH public key pair

Create another set of SSH public key pair. This key will be used for compute instance SSH access. We will be specifying public key will creating the instance. Private key will be specified while connecting to the instance.

        # ssh-keygen
        Generating public/private rsa key pair.
        Enter file in which to save the key (/root/.ssh/id_rsa):
        Enter passphrase (empty for no passphrase):
        Enter same passphrase again:
        Your identification has been saved in /root/.ssh/id_rsa.
        Your public key has been saved in /root/.ssh/id_rsa.pub.
        The key fingerprint is:
        SHA256:2rVDTujOKBMWspEdFCKg8/omlPVhFOYRsZ/T+nDunHk root@terraform
        The key's randomart image is:
        +---[RSA 2048]----+
        |+ ..oB+          |
        |.. .+.o          |
        |o  o.+           |
        | o+.oo. o.       |
        |  ++o..+S.+      |
        | +. o. +o= .     |
        |o  . ..oo.+      |
        |... o  +* oE     |
        | o.  o. +B.      |
        +----[SHA256]-----+
        # ls -lrt /root/.ssh/
        total 12
        -rw-------. 1 root root  550 Jan 29 09:27 authorized_keys
        -rw-------. 1 root root 1679 Feb 17 06:56 id_rsa
        -rw-r--r--. 1 root root  396 Feb 17 06:56 id_rsa.pub


#### Region
Region at which OCI account is associated. You can find this information in the console easily

#### Compartment OCID
Open the navigation menu, under Identity you can find compartments. Click that. It will list all the compartments available with the account. Choose the compartment on which we need to create instance. That will show the compartment details including OCID as shown in the picture.

![Compartment](https://github.com/kmkittu/OKE/blob/main/Compartment%20OCID.png)

## 2) Download the terraform script and extract it 

Download OKE.zip file https://github.com/kmkittu/OKE/blob/main/OKE.zip
Extract the zip file. It will create the below files.
template.tfvars
kube_config.tf
provider.tf
vcn.tf
cluster.tf
Cluster_kubeconfig
terraform.tfstate

Edit vcn.tf file and modify the below attributes with above collected values.

        variable "compartment_name" { default = "Terraform"}
        variable "compartment_ocid" {default = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"}

        variable vcn_cidr_prefix { default = "10.0" }  ===> Specify CIDR prefix values for VCN
        variable "vcn_cidr" {
        default = "10.0.0.0/16"                      ===> Specify CIDR prefix values for VCN
        }

        variable "user_ocid" { default = "ocid1.user.oc1..aaaaaaaan6dokkau7zyiemiggmkht2vvkvhgij3usqnhms5jvddm6h6tbfza"}
        variable "fingerprint" { default = "2f:80:77:4c:30:8c:5c:e7:d9:94:68:f3:db:3a:ab:25"}
        variable "region" {default = "ap-mumbai-1"}
        variable "private_key_path" { default = "/root/.oci/oci_api_key.pem"}  ===> Specify private key created earlier along with path
        

## 3) Check the integrity of the code 

Execute "terraform plan" command. OKE cluster will create instances. The plan command will ask for Public key which will be used by all cluster instances. Provide the SSH public key (id_rsa.pub) that we had created earlier.

        # terraform plan
        var.node_pool_ssh_public_key
        Enter a value: ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAhqXpiwufmWCWjfP3r44hKOXeQut7sj7yRDbJW+dTwYL3ynOksAEBGRTsXAPtEc3s/btavVwhT+O5GwsPHKv3g1uPD4yw33wG+/aSFZ3IV1F1Vi0RG5ipHSlwecgmaoNGP2E3ZJnvG3Glp3wkERI+K9RusuvrpxxysQrGrhv0yLzFl55iE3hT2HUtKef/QVk9eSltKyTYFAHQRxSaBnoo10K6zVSuHet5PHA8FkmXgdho77weHi14L7fH+exKXscFSmznNF8YCeOe1hTNWbFMFT5SiLVuv2NHGOV/6eJqtk6qWhHHj3yZVUv/jKtfs2eTU+HsgsxBGKhiebzo598T8w== rsa-key-20170125

        oci_core_virtual_network.oke-vcn: Refreshing state... [id=ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q]
        oci_containerengine_cluster.tfsample_cluster: Refreshing state... [id=ocid1.cluster.oc1.ap-mumbai-1.aaaaaaaaae4dooddmfqtezbyg4zwkzjzg44tqy3fg44tkndbhcqtgzbzhezg]
        oci_core_security_list.oke-lb-security-list: Refreshing state... [id=ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaajynlozl5x27qrl4xdu63kgg5dxbpp7fvym6x2kxukimayk6fpjga]
        oci_core_default_security_list.oke-default-security-list: Refreshing state... [id=ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaamiik64dj5xmz7vwup37ubbomnjoiwq6vicn2yuucne4zkbt6ckza]
        oci_core_security_list.oke-worker-security-list: Refreshing state... [id=ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaavyp2tgcfztdk32xmo7psizye3s46ena6f7yacikqvyx2u43b35cq]
        oci_core_internet_gateway.oke-igateway: Refreshing state... [id=ocid1.internetgateway.oc1.ap-mumbai-1.aaaaaaaamrtbbioum5rg335fabhlbprkagp5bdjchoibsr6vfy6eb7q7lm2a]
        oci_core_default_dhcp_options.oke-default-dhcp-options: Refreshing state... [id=ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja]
        oci_core_default_route_table.oke-default-route-table: Refreshing state... [id=ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq]
        oci_core_subnet.oke-subnet-loadbalancer-1: Refreshing state... [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaabbs5k5jze7tm7vkqrllikss4eaixxx4yzwzlomhodf3c2iqzpdgq]
        oci_core_subnet.oke-subnet-loadbalancer-2: Refreshing state... [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaavrzf3cvidoe7cyt7nc5y5ahe2tk54rqtnss2ydzrjmeybkmgcdvq]
        oci_core_subnet.oke-subnet-worker-2: Refreshing state... [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaajmmocnjfattmmu6i73lbnxa4tzt2bmtgarxbsifkzy6tuw5bumka]
        oci_core_subnet.oke-subnet-worker-3: Refreshing state... [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaaeqse6vdtphn32zfwzmrepcs2xryv5asla3m3ec5smjp4rhtdtniq]
        oci_core_subnet.oke-subnet-worker-1: Refreshing state... [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaamoj23v524jprabzpoqb2fwsz5f7hfn7d7sm3tx53e5ah3w2fbigq]
        oci_containerengine_node_pool.tfsample_node_pool: Refreshing state... [id=ocid1.nodepool.oc1.ap-mumbai-1.aaaaaaaaae4wky3fguzdqnrvmu2dqmddgrtdanlghe3ggodbgnywmmjwgezd]
        local_file.tfsample_cluster_kube_config_file: Refreshing state... [id=daf23dfb214054ccabfa9bb83a8bacae054110ed]

        An execution plan has been generated and is shown below.
        Resource actions are indicated with the following symbols:

        Terraform will perform the following actions:

        Plan: 0 to add, 0 to change, 0 to destroy.

        Changes to Outputs:
        ~ DHCPOptions = [
            ~ {
                ~ display_name   = "Default DHCP Options for Cluster_vcn" -> "Cluster-default-dhcp-options"
                ~ options        = [
                        {
                            custom_dns_servers  = []
                            search_domain_names = []
                            server_type         = "VcnLocalPlusInternet"
                            type                = "DomainNameServer"
                        },
                    - {
                        - custom_dns_servers  = []
                        - search_domain_names = [
                            - "clustervcn.oraclevcn.com",
                            ]
                        - server_type         = ""
                        - type                = "SearchDomain"
                        },
                    ]
                    # (7 unchanged elements hidden)
                },
            ]
        ~ RouteTables = [
            ~ {
                ~ display_name   = "Default Route Table for Cluster_vcn" -> "Cluster-default-route-table"
                ~ route_rules    = [
                    + {
                        + cidr_block        = "0.0.0.0/0"
                        + description       = ""
                        + destination       = "0.0.0.0/0"
                        + destination_type  = "CIDR_BLOCK"
                        + network_entity_id = "ocid1.internetgateway.oc1.ap-mumbai-1.aaaaaaaamrtbbioum5rg335fabhlbprkagp5bdjchoibsr6vfy6eb7q7lm2a"
                        },
                    ]
                    # (7 unchanged elements hidden)
                },
            ]
        ~ Subnets     = [
            ~ {
                ~ state                      = "PROVISIONING" -> "AVAILABLE"
                    # (20 unchanged elements hidden)
                },
                {
                    availability_domain        = "LxbT:AP-MUMBAI-1-AD-1"
                    cidr_block                 = "10.0.10.0/24"
                    compartment_id             = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
                    defined_tags               = {
                        "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
                        "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:23.559Z"
                    }
                    dhcp_options_id            = "ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja"
                    display_name               = "Cluster-WorkerSubnet01"
                    dns_label                  = "workers01"
                    freeform_tags              = {}
                    id                         = "ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaamoj23v524jprabzpoqb2fwsz5f7hfn7d7sm3tx53e5ah3w2fbigq"
                    ipv6cidr_block             = ""
                    ipv6public_cidr_block      = ""
                    ipv6virtual_router_ip      = ""
                    prohibit_public_ip_on_vnic = false
                    route_table_id             = "ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq"
                    security_list_ids          = [
                        "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaavyp2tgcfztdk32xmo7psizye3s46ena6f7yacikqvyx2u43b35cq",
                    ]
                    state                      = "AVAILABLE"
                    subnet_domain_name         = "workers01.clustervcn.oraclevcn.com"
                    time_created               = "2021-02-16 10:55:23.97 +0000 UTC"
                    vcn_id                     = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
                    virtual_router_ip          = "10.0.10.1"
                    virtual_router_mac         = "00:00:17:CA:BC:FE"
                },
                # (3 unchanged elements hidden)
            ]

        Warning: Interpolation-only expressions are deprecated

        on cluster.tf line 19, in data "oci_identity_availability_domains" "tfsample_availability_domains":
        19:   compartment_id = "${var.compartment_ocid}"

        Terraform 0.11 and earlier required all non-constant expressions to be
        provided via interpolation syntax, but this pattern is now deprecated. To
        silence this warning, remove the "${ sequence from the start and the }"
        sequence from the end of this expression, leaving just the inner expression.

        Template interpolation syntax is still used to construct strings from
        expressions when the template includes multiple interpolation sequences or a
        mixture of literal strings and interpolations. This deprecation applies only
        to templates that consist entirely of a single interpolation sequence.

        (and 84 more similar warnings elsewhere)


        Warning: "node_image_name": [DEPRECATED] The 'node_image_name' field has been deprecated. Please use 'node_source_details' instead. If both fields are specified, then 'node_source_details' will be used.

        on cluster.tf line 43, in resource "oci_containerengine_node_pool" "tfsample_node_pool":
        43: resource "oci_containerengine_node_pool" "tfsample_node_pool" {



        Warning: Version constraints inside provider configuration blocks are deprecated

        on provider.tf line 15, in provider "oci":
        15:   version          = ">= 3.0.0"

        Terraform 0.13 and earlier allowed provider version constraints inside the
        provider configuration block, but that is now deprecated and will be removed
        in a future version of Terraform. To silence this warning, move the provider
        version constraint into the required_providers block.


        ------------------------------------------------------------------------

        Note: You didn't specify an "-out" parameter to save this plan, so Terraform
        can't guarantee that exactly these actions will be performed if
        "terraform apply" is subsequently run.





## 4) Execute the Terraform script 

Execute "terraform apply" command. It will as for the SSH public key which will be used by cluster instances.

        [root@terraform oke]# terraform apply -auto-approve
        var.node_pool_ssh_public_key
        Enter a value: ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAhqXpiwufmWCWjfP3r44hKOXeQut7sj7yRDbJW+dTwYL3ynOksAEBGRTsXAPtEc3s/btavVwhT+O5GwsPHKv3g1uPD4yw33wG+/aSFZ3IV1F1Vi0RG5ipHSlwecgmaoNGP2E3ZJnvG3Glp3wkERI+K9RusuvrpxxysQrGrhv0yLzFl55iE3hT2HUtKef/QVk9eSltKyTYFAHQRxSaBnoo10K6zVSuHet5PHA8FkmXgdho77weHi14L7fH+exKXscFSmznNF8YCeOe1hTNWbFMFT5SiLVuv2NHGOV/6eJqtk6qWhHHj3yZVUv/jKtfs2eTU+HsgsxBGKhiebzo598T8w== rsa-key-20170125


        An execution plan has been generated and is shown below.
        Resource actions are indicated with the following symbols:
        + create
        <= read (data resources)

        Terraform will perform the following actions:

        # data.oci_containerengine_cluster_kube_config.tfsample_cluster_kube_config will be read during apply
        # (config refers to values not yet known)
        <= data "oci_containerengine_cluster_kube_config" "tfsample_cluster_kube_config"  {
            + cluster_id = (known after apply)
            + content    = (known after apply)
            + id         = (known after apply)
            }

        # data.oci_core_dhcp_options.oke_dhcp_options will be read during apply
        # (config refers to values not yet known)
        <= data "oci_core_dhcp_options" "oke_dhcp_options"  {
            + compartment_id = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + id             = (known after apply)
            + options        = (known after apply)
            + vcn_id         = (known after apply)
            }

        # data.oci_core_internet_gateways.oke-igateways will be read during apply
        # (config refers to values not yet known)
        <= data "oci_core_internet_gateways" "oke-igateways"  {
            + compartment_id = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + gateways       = (known after apply)
            + id             = (known after apply)
            + vcn_id         = (known after apply)
            }

        # data.oci_core_route_tables.oke_route_tables will be read during apply
        # (config refers to values not yet known)
        <= data "oci_core_route_tables" "oke_route_tables"  {
            + compartment_id = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + id             = (known after apply)
            + route_tables   = (known after apply)
            + vcn_id         = (known after apply)
            }

        # data.oci_core_security_lists.oke_security_lists will be read during apply
        # (config refers to values not yet known)
        <= data "oci_core_security_lists" "oke_security_lists"  {
            + compartment_id = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + id             = (known after apply)
            + security_lists = (known after apply)
            + vcn_id         = (known after apply)
            }

        # data.oci_core_subnets.oke_subnets will be read during apply
        # (config refers to values not yet known)
        <= data "oci_core_subnets" "oke_subnets"  {
            + compartment_id = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + id             = (known after apply)
            + subnets        = (known after apply)
            + vcn_id         = (known after apply)
            }

        # data.oci_core_virtual_networks.oke-vcns will be read during apply
        # (config refers to values not yet known)
        <= data "oci_core_virtual_networks" "oke-vcns"  {
            + compartment_id   = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + display_name     = "Cluster_vcn"
            + id               = (known after apply)
            + virtual_networks = (known after apply)
            }

        # local_file.tfsample_cluster_kube_config_file will be created
        + resource "local_file" "tfsample_cluster_kube_config_file" {
            + content              = (known after apply)
            + directory_permission = "0777"
            + file_permission      = "0777"
            + filename             = "./Cluster_kubeconfig"
            + id                   = (known after apply)
            }

        # oci_containerengine_cluster.tfsample_cluster will be created
        + resource "oci_containerengine_cluster" "tfsample_cluster" {
            + available_kubernetes_upgrades = (known after apply)
            + compartment_id                = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + endpoints                     = (known after apply)
            + id                            = (known after apply)
            + kms_key_id                    = (known after apply)
            + kubernetes_version            = "v1.18.10"
            + lifecycle_details             = (known after apply)
            + metadata                      = (known after apply)
            + name                          = "Cluster"
            + state                         = (known after apply)
            + vcn_id                        = (known after apply)

            + options {
                + service_lb_subnet_ids = (known after apply)

                + add_ons {
                    + is_kubernetes_dashboard_enabled = true
                    + is_tiller_enabled               = true
                    }

                + admission_controller_options {
                    + is_pod_security_policy_enabled = (known after apply)
                    }

                + kubernetes_network_config {
                    + pods_cidr     = (known after apply)
                    + services_cidr = (known after apply)
                    }
                }
            }

        # oci_containerengine_node_pool.tfsample_node_pool will be created
        + resource "oci_containerengine_node_pool" "tfsample_node_pool" {
            + cluster_id          = (known after apply)
            + compartment_id      = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + id                  = (known after apply)
            + kubernetes_version  = "v1.18.10"
            + name                = "tfTestCluster_workers"
            + node_image_id       = (known after apply)
            + node_image_name     = "Oracle-Linux-7.6"
            + node_metadata       = (known after apply)
            + node_shape          = "VM.Standard2.1"
            + node_source         = (known after apply)
            + nodes               = (known after apply)
            + quantity_per_subnet = 2
            + ssh_public_key      = "ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAhqXpiwufmWCWjfP3r44hKOXeQut7sj7yRDbJW+dTwYL3ynOksAEBGRTsXAPtEc3s/btavVwhT+O5GwsPHKv3g1uPD4yw33wG+/aSFZ3IV1F1Vi0RG5ipHSlwecgmaoNGP2E3ZJnvG3Glp3wkERI+K9RusuvrpxxysQrGrhv0yLzFl55iE3hT2HUtKef/QVk9eSltKyTYFAHQRxSaBnoo10K6zVSuHet5PHA8FkmXgdho77weHi14L7fH+exKXscFSmznNF8YCeOe1hTNWbFMFT5SiLVuv2NHGOV/6eJqtk6qWhHHj3yZVUv/jKtfs2eTU+HsgsxBGKhiebzo598T8w== rsa-key-20170125"
            + subnet_ids          = (known after apply)

            + initial_node_labels {
                + key   = (known after apply)
                + value = (known after apply)
                }

            + node_config_details {
                + size = (known after apply)

                + placement_configs {
                    + availability_domain = (known after apply)
                    + subnet_id           = (known after apply)
                    }
                }

            + node_shape_config {
                + memory_in_gbs = (known after apply)
                + ocpus         = (known after apply)
                }

            + node_source_details {
                + boot_volume_size_in_gbs = (known after apply)
                + image_id                = (known after apply)
                + source_type             = (known after apply)
                }
            }

        # oci_core_default_dhcp_options.oke-default-dhcp-options will be created
        + resource "oci_core_default_dhcp_options" "oke-default-dhcp-options" {
            + defined_tags               = (known after apply)
            + display_name               = "Cluster-default-dhcp-options"
            + freeform_tags              = (known after apply)
            + id                         = (known after apply)
            + manage_default_resource_id = (known after apply)
            + state                      = (known after apply)
            + time_created               = (known after apply)

            + options {
                + custom_dns_servers  = []
                + search_domain_names = (known after apply)
                + server_type         = "VcnLocalPlusInternet"
                + type                = "DomainNameServer"
                }
            }

        # oci_core_default_route_table.oke-default-route-table will be created
        + resource "oci_core_default_route_table" "oke-default-route-table" {
            + defined_tags               = (known after apply)
            + display_name               = "Cluster-default-route-table"
            + freeform_tags              = (known after apply)
            + id                         = (known after apply)
            + manage_default_resource_id = (known after apply)
            + state                      = (known after apply)
            + time_created               = (known after apply)

            + route_rules {
                + cidr_block        = "0.0.0.0/0"
                + description       = (known after apply)
                + destination       = (known after apply)
                + destination_type  = (known after apply)
                + network_entity_id = (known after apply)
                }
            }

        # oci_core_default_security_list.oke-default-security-list will be created
        + resource "oci_core_default_security_list" "oke-default-security-list" {
            + defined_tags               = (known after apply)
            + display_name               = "Cluster-default-security-list"
            + freeform_tags              = (known after apply)
            + id                         = (known after apply)
            + manage_default_resource_id = (known after apply)
            + state                      = (known after apply)
            + time_created               = (known after apply)

            + egress_security_rules {
                + description      = (known after apply)
                + destination      = "0.0.0.0/0"
                + destination_type = (known after apply)
                + protocol         = "all"
                + stateless        = (known after apply)
                }

            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "1"
                + source      = "0.0.0.0/0"
                + source_type = (known after apply)
                + stateless   = false

                + icmp_options {
                    + code = 4
                    + type = 3
                    }
                }
            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "6"
                + source      = "0.0.0.0/0"
                + source_type = (known after apply)
                + stateless   = false

                + tcp_options {
                    + max = 22
                    + min = 22
                    }
                }
            }

        # oci_core_internet_gateway.oke-igateway will be created
        + resource "oci_core_internet_gateway" "oke-igateway" {
            + compartment_id = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + defined_tags   = (known after apply)
            + display_name   = "Cluster-igateway"
            + enabled        = true
            + freeform_tags  = (known after apply)
            + id             = (known after apply)
            + state          = (known after apply)
            + time_created   = (known after apply)
            + vcn_id         = (known after apply)
            }

        # oci_core_security_list.oke-lb-security-list will be created
        + resource "oci_core_security_list" "oke-lb-security-list" {
            + compartment_id = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + defined_tags   = (known after apply)
            + display_name   = "Cluster-LoadBalancers-SecList"
            + freeform_tags  = (known after apply)
            + id             = (known after apply)
            + state          = (known after apply)
            + time_created   = (known after apply)
            + vcn_id         = (known after apply)

            + egress_security_rules {
                + description      = (known after apply)
                + destination      = "0.0.0.0/0"
                + destination_type = (known after apply)
                + protocol         = "6"
                + stateless        = true
                }

            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "6"
                + source      = "0.0.0.0/0"
                + source_type = (known after apply)
                + stateless   = true
                }
            }

        # oci_core_security_list.oke-worker-security-list will be created
        + resource "oci_core_security_list" "oke-worker-security-list" {
            + compartment_id = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + defined_tags   = (known after apply)
            + display_name   = "Cluster-Workers-SecList"
            + freeform_tags  = (known after apply)
            + id             = (known after apply)
            + state          = (known after apply)
            + time_created   = (known after apply)
            + vcn_id         = (known after apply)

            + egress_security_rules {
                + description      = (known after apply)
                + destination      = "0.0.0.0/0"
                + destination_type = (known after apply)
                + protocol         = "6"
                + stateless        = false
                }
            + egress_security_rules {
                + description      = (known after apply)
                + destination      = "10.0.10.0/24"
                + destination_type = (known after apply)
                + protocol         = "all"
                + stateless        = true
                }
            + egress_security_rules {
                + description      = (known after apply)
                + destination      = "10.0.11.0/24"
                + destination_type = (known after apply)
                + protocol         = "all"
                + stateless        = true
                }
            + egress_security_rules {
                + description      = (known after apply)
                + destination      = "10.0.12.0/24"
                + destination_type = (known after apply)
                + protocol         = "all"
                + stateless        = true
                }

            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "1"
                + source      = "0.0.0.0/0"
                + source_type = (known after apply)
                + stateless   = false

                + icmp_options {
                    + code = 4
                    + type = 3
                    }
                }
            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "6"
                + source      = "0.0.0.0/0"
                + source_type = (known after apply)
                + stateless   = false

                + tcp_options {
                    + max = 22
                    + min = 22
                    }
                }
            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "6"
                + source      = "0.0.0.0/0"
                + source_type = (known after apply)
                + stateless   = true

                + tcp_options {
                    + max = 32767
                    + min = 30000
                    }
                }
            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "6"
                + source      = "130.35.0.0/16"
                + source_type = (known after apply)
                + stateless   = false

                + tcp_options {
                    + max = 22
                    + min = 22
                    }
                }
            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "6"
                + source      = "138.1.0.0/17"
                + source_type = (known after apply)
                + stateless   = false

                + tcp_options {
                    + max = 22
                    + min = 22
                    }
                }
            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "all"
                + source      = "10.0.10.0/24"
                + source_type = (known after apply)
                + stateless   = true
                }
            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "all"
                + source      = "10.0.11.0/24"
                + source_type = (known after apply)
                + stateless   = true
                }
            + ingress_security_rules {
                + description = (known after apply)
                + protocol    = "all"
                + source      = "10.0.12.0/24"
                + source_type = (known after apply)
                + stateless   = true
                }
            }

        # oci_core_subnet.oke-subnet-loadbalancer-1 will be created
        + resource "oci_core_subnet" "oke-subnet-loadbalancer-1" {
            + availability_domain        = "LxbT:AP-MUMBAI-1-AD-1"
            + cidr_block                 = "10.0.20.0/24"
            + compartment_id             = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + defined_tags               = (known after apply)
            + dhcp_options_id            = (known after apply)
            + display_name               = "Cluster-LB-Subnet01"
            + dns_label                  = "lb01"
            + freeform_tags              = (known after apply)
            + id                         = (known after apply)
            + ipv6cidr_block             = (known after apply)
            + ipv6public_cidr_block      = (known after apply)
            + ipv6virtual_router_ip      = (known after apply)
            + prohibit_public_ip_on_vnic = (known after apply)
            + route_table_id             = (known after apply)
            + security_list_ids          = (known after apply)
            + state                      = (known after apply)
            + subnet_domain_name         = (known after apply)
            + time_created               = (known after apply)
            + vcn_id                     = (known after apply)
            + virtual_router_ip          = (known after apply)
            + virtual_router_mac         = (known after apply)
            }

        # oci_core_subnet.oke-subnet-loadbalancer-2 will be created
        + resource "oci_core_subnet" "oke-subnet-loadbalancer-2" {
            + availability_domain        = "LxbT:AP-MUMBAI-1-AD-1"
            + cidr_block                 = "10.0.21.0/24"
            + compartment_id             = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + defined_tags               = (known after apply)
            + dhcp_options_id            = (known after apply)
            + display_name               = "Cluster-LB-Subnet02"
            + dns_label                  = "lb02"
            + freeform_tags              = (known after apply)
            + id                         = (known after apply)
            + ipv6cidr_block             = (known after apply)
            + ipv6public_cidr_block      = (known after apply)
            + ipv6virtual_router_ip      = (known after apply)
            + prohibit_public_ip_on_vnic = (known after apply)
            + route_table_id             = (known after apply)
            + security_list_ids          = (known after apply)
            + state                      = (known after apply)
            + subnet_domain_name         = (known after apply)
            + time_created               = (known after apply)
            + vcn_id                     = (known after apply)
            + virtual_router_ip          = (known after apply)
            + virtual_router_mac         = (known after apply)
            }

        # oci_core_subnet.oke-subnet-worker-1 will be created
        + resource "oci_core_subnet" "oke-subnet-worker-1" {
            + availability_domain        = "LxbT:AP-MUMBAI-1-AD-1"
            + cidr_block                 = "10.0.10.0/24"
            + compartment_id             = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + defined_tags               = (known after apply)
            + dhcp_options_id            = (known after apply)
            + display_name               = "Cluster-WorkerSubnet01"
            + dns_label                  = "workers01"
            + freeform_tags              = (known after apply)
            + id                         = (known after apply)
            + ipv6cidr_block             = (known after apply)
            + ipv6public_cidr_block      = (known after apply)
            + ipv6virtual_router_ip      = (known after apply)
            + prohibit_public_ip_on_vnic = (known after apply)
            + route_table_id             = (known after apply)
            + security_list_ids          = (known after apply)
            + state                      = (known after apply)
            + subnet_domain_name         = (known after apply)
            + time_created               = (known after apply)
            + vcn_id                     = (known after apply)
            + virtual_router_ip          = (known after apply)
            + virtual_router_mac         = (known after apply)
            }

        # oci_core_subnet.oke-subnet-worker-2 will be created
        + resource "oci_core_subnet" "oke-subnet-worker-2" {
            + availability_domain        = "LxbT:AP-MUMBAI-1-AD-1"
            + cidr_block                 = "10.0.11.0/24"
            + compartment_id             = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + defined_tags               = (known after apply)
            + dhcp_options_id            = (known after apply)
            + display_name               = "Cluster-WorkerSubnet02"
            + dns_label                  = "workers02"
            + freeform_tags              = (known after apply)
            + id                         = (known after apply)
            + ipv6cidr_block             = (known after apply)
            + ipv6public_cidr_block      = (known after apply)
            + ipv6virtual_router_ip      = (known after apply)
            + prohibit_public_ip_on_vnic = (known after apply)
            + route_table_id             = (known after apply)
            + security_list_ids          = (known after apply)
            + state                      = (known after apply)
            + subnet_domain_name         = (known after apply)
            + time_created               = (known after apply)
            + vcn_id                     = (known after apply)
            + virtual_router_ip          = (known after apply)
            + virtual_router_mac         = (known after apply)
            }

        # oci_core_subnet.oke-subnet-worker-3 will be created
        + resource "oci_core_subnet" "oke-subnet-worker-3" {
            + availability_domain        = "LxbT:AP-MUMBAI-1-AD-1"
            + cidr_block                 = "10.0.12.0/24"
            + compartment_id             = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + defined_tags               = (known after apply)
            + dhcp_options_id            = (known after apply)
            + display_name               = "Cluster-WorkerSubnet03"
            + dns_label                  = "workers03"
            + freeform_tags              = (known after apply)
            + id                         = (known after apply)
            + ipv6cidr_block             = (known after apply)
            + ipv6public_cidr_block      = (known after apply)
            + ipv6virtual_router_ip      = (known after apply)
            + prohibit_public_ip_on_vnic = (known after apply)
            + route_table_id             = (known after apply)
            + security_list_ids          = (known after apply)
            + state                      = (known after apply)
            + subnet_domain_name         = (known after apply)
            + time_created               = (known after apply)
            + vcn_id                     = (known after apply)
            + virtual_router_ip          = (known after apply)
            + virtual_router_mac         = (known after apply)
            }

        # oci_core_virtual_network.oke-vcn will be created
        + resource "oci_core_virtual_network" "oke-vcn" {
            + cidr_block               = "10.0.0.0/16"
            + cidr_blocks              = (known after apply)
            + compartment_id           = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            + default_dhcp_options_id  = (known after apply)
            + default_route_table_id   = (known after apply)
            + default_security_list_id = (known after apply)
            + defined_tags             = (known after apply)
            + display_name             = "Cluster_vcn"
            + dns_label                = "Clustervcn"
            + freeform_tags            = (known after apply)
            + id                       = (known after apply)
            + ipv6cidr_block           = (known after apply)
            + ipv6public_cidr_block    = (known after apply)
            + is_ipv6enabled           = (known after apply)
            + state                    = (known after apply)
            + time_created             = (known after apply)
            + vcn_domain_name          = (known after apply)
            }

        Plan: 15 to add, 0 to change, 0 to destroy.

        Changes to Outputs:
        + Compartments    = []
        + DHCPOptions     = (known after apply)
        + InternetGateway = (known after apply)
        + RouteTables     = (known after apply)
        + SecurityLists   = (known after apply)
        + Subnets         = (known after apply)
        + VCN             = (known after apply)
        + cluster_id      = (known after apply)


        Warning: Interpolation-only expressions are deprecated

        on cluster.tf line 19, in data "oci_identity_availability_domains" "tfsample_availability_domains":
        19:   compartment_id = "${var.compartment_ocid}"

        Terraform 0.11 and earlier required all non-constant expressions to be
        provided via interpolation syntax, but this pattern is now deprecated. To
        silence this warning, remove the "${ sequence from the start and the }"
        sequence from the end of this expression, leaving just the inner expression.

        Template interpolation syntax is still used to construct strings from
        expressions when the template includes multiple interpolation sequences or a
        mixture of literal strings and interpolations. This deprecation applies only
        to templates that consist entirely of a single interpolation sequence.

        (and 84 more similar warnings elsewhere)


        Warning: "node_image_name": [DEPRECATED] The 'node_image_name' field has been deprecated. Please use 'node_source_details' instead. If both fields are specified, then 'node_source_details' will be used.

        on cluster.tf line 43, in resource "oci_containerengine_node_pool" "tfsample_node_pool":
        43: resource "oci_containerengine_node_pool" "tfsample_node_pool" {



        Warning: Version constraints inside provider configuration blocks are deprecated

        on provider.tf line 15, in provider "oci":
        15:   version          = ">= 3.0.0"

        Terraform 0.13 and earlier allowed provider version constraints inside the
        provider configuration block, but that is now deprecated and will be removed
        in a future version of Terraform. To silence this warning, move the provider
        version constraint into the required_providers block.

        oci_core_virtual_network.oke-vcn: Creating...
        oci_core_virtual_network.oke-vcn: Creation complete after 0s [id=ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q]
        data.oci_core_route_tables.oke_route_tables: Reading...
        data.oci_core_dhcp_options.oke_dhcp_options: Reading...
        data.oci_core_virtual_networks.oke-vcns: Reading...
        oci_core_internet_gateway.oke-igateway: Creating...
        oci_core_default_dhcp_options.oke-default-dhcp-options: Creating...
        oci_core_default_security_list.oke-default-security-list: Creating...
        oci_core_security_list.oke-lb-security-list: Creating...
        oci_containerengine_cluster.tfsample_cluster: Creating...
        data.oci_core_route_tables.oke_route_tables: Read complete after 1s [id=CoreRouteTablesDataSource-3809201265]
        oci_core_security_list.oke-worker-security-list: Creating...
        data.oci_core_dhcp_options.oke_dhcp_options: Read complete after 1s [id=CoreDhcpOptionsDataSource-3809201265]
        data.oci_core_virtual_networks.oke-vcns: Read complete after 0s [id=CoreVcnsDataSource-1777932344]
        oci_core_default_dhcp_options.oke-default-dhcp-options: Creation complete after 0s [id=ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja]
        oci_core_internet_gateway.oke-igateway: Creation complete after 0s [id=ocid1.internetgateway.oc1.ap-mumbai-1.aaaaaaaamrtbbioum5rg335fabhlbprkagp5bdjchoibsr6vfy6eb7q7lm2a]
        data.oci_core_internet_gateways.oke-igateways: Reading...
        oci_core_default_route_table.oke-default-route-table: Creating...
        oci_core_security_list.oke-lb-security-list: Creation complete after 0s [id=ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaajynlozl5x27qrl4xdu63kgg5dxbpp7fvym6x2kxukimayk6fpjga]
        data.oci_core_internet_gateways.oke-igateways: Read complete after 0s [id=CoreInternetGatewaysDataSource-3809201265]
        oci_core_subnet.oke-subnet-loadbalancer-2: Creating...
        oci_core_subnet.oke-subnet-loadbalancer-1: Creating...
        oci_core_default_security_list.oke-default-security-list: Creation complete after 0s [id=ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaamiik64dj5xmz7vwup37ubbomnjoiwq6vicn2yuucne4zkbt6ckza]
        oci_core_security_list.oke-worker-security-list: Creation complete after 0s [id=ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaavyp2tgcfztdk32xmo7psizye3s46ena6f7yacikqvyx2u43b35cq]
        data.oci_core_security_lists.oke_security_lists: Reading...
        oci_core_subnet.oke-subnet-worker-3: Creating...
        oci_core_subnet.oke-subnet-worker-2: Creating...
        oci_core_subnet.oke-subnet-worker-1: Creating...
        oci_core_default_route_table.oke-default-route-table: Creation complete after 0s [id=ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq]
        data.oci_core_security_lists.oke_security_lists: Read complete after 0s [id=CoreSecurityListsDataSource-3809201265]
        oci_core_subnet.oke-subnet-loadbalancer-1: Creation complete after 7s [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaabbs5k5jze7tm7vkqrllikss4eaixxx4yzwzlomhodf3c2iqzpdgq]
        oci_core_subnet.oke-subnet-loadbalancer-2: Creation complete after 8s [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaavrzf3cvidoe7cyt7nc5y5ahe2tk54rqtnss2ydzrjmeybkmgcdvq]
        oci_core_subnet.oke-subnet-worker-2: Creation complete after 8s [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaajmmocnjfattmmu6i73lbnxa4tzt2bmtgarxbsifkzy6tuw5bumka]
        oci_core_subnet.oke-subnet-worker-1: Creation complete after 8s [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaamoj23v524jprabzpoqb2fwsz5f7hfn7d7sm3tx53e5ah3w2fbigq]
        data.oci_core_subnets.oke_subnets: Reading...
        data.oci_core_subnets.oke_subnets: Read complete after 0s [id=CoreSubnetsDataSource-3809201265]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [10s elapsed]
        oci_core_subnet.oke-subnet-worker-3: Still creating... [10s elapsed]
        oci_core_subnet.oke-subnet-worker-3: Creation complete after 15s [id=ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaaeqse6vdtphn32zfwzmrepcs2xryv5asla3m3ec5smjp4rhtdtniq]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [20s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [30s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [40s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [50s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [1m0s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [1m10s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [1m20s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [1m30s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [1m40s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [1m50s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [2m0s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [2m10s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [2m20s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [2m30s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [2m40s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [2m50s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [3m0s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [3m10s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [3m20s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [3m30s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [3m40s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [3m50s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [4m0s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Still creating... [4m10s elapsed]
        oci_containerengine_cluster.tfsample_cluster: Creation complete after 4m15s [id=ocid1.cluster.oc1.ap-mumbai-1.aaaaaaaaae4dooddmfqtezbyg4zwkzjzg44tqy3fg44tkndbhcqtgzbzhezg]
        data.oci_containerengine_cluster_kube_config.tfsample_cluster_kube_config: Reading...
        oci_containerengine_node_pool.tfsample_node_pool: Creating...
        data.oci_containerengine_cluster_kube_config.tfsample_cluster_kube_config: Read complete after 0s [id=ContainerengineClusterKubeConfigDataSource-2796868170]
        local_file.tfsample_cluster_kube_config_file: Creating...
        local_file.tfsample_cluster_kube_config_file: Creation complete after 0s [id=daf23dfb214054ccabfa9bb83a8bacae054110ed]
        oci_containerengine_node_pool.tfsample_node_pool: Creation complete after 2s [id=ocid1.nodepool.oc1.ap-mumbai-1.aaaaaaaaae4wky3fguzdqnrvmu2dqmddgrtdanlghe3ggodbgnywmmjwgezd]

        Apply complete! Resources: 15 added, 0 changed, 0 destroyed.

        Outputs:

        Compartments = tolist([])
        DHCPOptions = tolist([
        {
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:21.958Z"
            })
            "display_name" = "Default DHCP Options for Cluster_vcn"
            "freeform_tags" = tomap({})
            "id" = "ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja"
            "options" = tolist([
            {
                "custom_dns_servers" = tolist([])
                "search_domain_names" = tolist([])
                "server_type" = "VcnLocalPlusInternet"
                "type" = "DomainNameServer"
            },
            {
                "custom_dns_servers" = tolist([])
                "search_domain_names" = tolist([
                "clustervcn.oraclevcn.com",
                ])
                "server_type" = ""
                "type" = "SearchDomain"
            },
            ])
            "state" = "AVAILABLE"
            "time_created" = "2021-02-16 10:55:22.181 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
        },
        ])
        InternetGateway = tolist([
        {
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:22.690Z"
            })
            "display_name" = "Cluster-igateway"
            "enabled" = true
            "freeform_tags" = tomap({})
            "id" = "ocid1.internetgateway.oc1.ap-mumbai-1.aaaaaaaamrtbbioum5rg335fabhlbprkagp5bdjchoibsr6vfy6eb7q7lm2a"
            "state" = "AVAILABLE"
            "time_created" = "2021-02-16 10:55:22.731 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
        },
        ])
        RouteTables = tolist([
        {
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:21.958Z"
            })
            "display_name" = "Default Route Table for Cluster_vcn"
            "freeform_tags" = tomap({})
            "id" = "ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq"
            "route_rules" = tolist([])
            "state" = "AVAILABLE"
            "time_created" = "2021-02-16 10:55:22.064 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
        },
        ])
        SecurityLists = tolist([
        {
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:22.768Z"
            })
            "display_name" = "Cluster-Workers-SecList"
            "egress_security_rules" = tolist([
            {
                "description" = ""
                "destination" = "0.0.0.0/0"
                "destination_type" = "CIDR_BLOCK"
                "icmp_options" = tolist([])
                "protocol" = "6"
                "stateless" = false
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "destination" = "10.0.12.0/24"
                "destination_type" = "CIDR_BLOCK"
                "icmp_options" = tolist([])
                "protocol" = "all"
                "stateless" = true
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "destination" = "10.0.10.0/24"
                "destination_type" = "CIDR_BLOCK"
                "icmp_options" = tolist([])
                "protocol" = "all"
                "stateless" = true
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "destination" = "10.0.11.0/24"
                "destination_type" = "CIDR_BLOCK"
                "icmp_options" = tolist([])
                "protocol" = "all"
                "stateless" = true
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            ])
            "freeform_tags" = tomap({})
            "id" = "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaavyp2tgcfztdk32xmo7psizye3s46ena6f7yacikqvyx2u43b35cq"
            "ingress_security_rules" = tolist([
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "all"
                "source" = "10.0.11.0/24"
                "source_type" = "CIDR_BLOCK"
                "stateless" = true
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "icmp_options" = tolist([
                {
                    "code" = 4
                    "type" = 3
                },
                ])
                "protocol" = "1"
                "source" = "0.0.0.0/0"
                "source_type" = "CIDR_BLOCK"
                "stateless" = false
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "6"
                "source" = "130.35.0.0/16"
                "source_type" = "CIDR_BLOCK"
                "stateless" = false
                "tcp_options" = tolist([
                {
                    "max" = 22
                    "min" = 22
                    "source_port_range" = tolist([])
                },
                ])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "all"
                "source" = "10.0.10.0/24"
                "source_type" = "CIDR_BLOCK"
                "stateless" = true
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "6"
                "source" = "0.0.0.0/0"
                "source_type" = "CIDR_BLOCK"
                "stateless" = true
                "tcp_options" = tolist([
                {
                    "max" = 32767
                    "min" = 30000
                    "source_port_range" = tolist([])
                },
                ])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "6"
                "source" = "138.1.0.0/17"
                "source_type" = "CIDR_BLOCK"
                "stateless" = false
                "tcp_options" = tolist([
                {
                    "max" = 22
                    "min" = 22
                    "source_port_range" = tolist([])
                },
                ])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "6"
                "source" = "0.0.0.0/0"
                "source_type" = "CIDR_BLOCK"
                "stateless" = false
                "tcp_options" = tolist([
                {
                    "max" = 22
                    "min" = 22
                    "source_port_range" = tolist([])
                },
                ])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "all"
                "source" = "10.0.12.0/24"
                "source_type" = "CIDR_BLOCK"
                "stateless" = true
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            ])
            "state" = "AVAILABLE"
            "time_created" = "2021-02-16 10:55:22.805 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
        },
        {
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:22.756Z"
            })
            "display_name" = "Cluster-LoadBalancers-SecList"
            "egress_security_rules" = tolist([
            {
                "description" = ""
                "destination" = "0.0.0.0/0"
                "destination_type" = "CIDR_BLOCK"
                "icmp_options" = tolist([])
                "protocol" = "6"
                "stateless" = true
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            ])
            "freeform_tags" = tomap({})
            "id" = "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaajynlozl5x27qrl4xdu63kgg5dxbpp7fvym6x2kxukimayk6fpjga"
            "ingress_security_rules" = tolist([
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "6"
                "source" = "0.0.0.0/0"
                "source_type" = "CIDR_BLOCK"
                "stateless" = true
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            ])
            "state" = "AVAILABLE"
            "time_created" = "2021-02-16 10:55:22.79 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
        },
        {
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:21.958Z"
            })
            "display_name" = "Cluster-default-security-list"
            "egress_security_rules" = tolist([
            {
                "description" = ""
                "destination" = "0.0.0.0/0"
                "destination_type" = "CIDR_BLOCK"
                "icmp_options" = tolist([])
                "protocol" = "all"
                "stateless" = false
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            ])
            "freeform_tags" = tomap({})
            "id" = "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaamiik64dj5xmz7vwup37ubbomnjoiwq6vicn2yuucne4zkbt6ckza"
            "ingress_security_rules" = tolist([
            {
                "description" = ""
                "icmp_options" = tolist([
                {
                    "code" = 4
                    "type" = 3
                },
                ])
                "protocol" = "1"
                "source" = "0.0.0.0/0"
                "source_type" = "CIDR_BLOCK"
                "stateless" = false
                "tcp_options" = tolist([])
                "udp_options" = tolist([])
            },
            {
                "description" = ""
                "icmp_options" = tolist([])
                "protocol" = "6"
                "source" = "0.0.0.0/0"
                "source_type" = "CIDR_BLOCK"
                "stateless" = false
                "tcp_options" = tolist([
                {
                    "max" = 22
                    "min" = 22
                    "source_port_range" = tolist([])
                },
                ])
                "udp_options" = tolist([])
            },
            ])
            "state" = "AVAILABLE"
            "time_created" = "2021-02-16 10:55:22.064 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
        },
        ])
        Subnets = tolist([
        {
            "availability_domain" = "LxbT:AP-MUMBAI-1-AD-1"
            "cidr_block" = "10.0.12.0/24"
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:23.595Z"
            })
            "dhcp_options_id" = "ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja"
            "display_name" = "Cluster-WorkerSubnet03"
            "dns_label" = "workers03"
            "freeform_tags" = tomap({})
            "id" = "ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaaeqse6vdtphn32zfwzmrepcs2xryv5asla3m3ec5smjp4rhtdtniq"
            "ipv6cidr_block" = ""
            "ipv6public_cidr_block" = ""
            "ipv6virtual_router_ip" = ""
            "prohibit_public_ip_on_vnic" = false
            "route_table_id" = "ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq"
            "security_list_ids" = tolist([
            "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaavyp2tgcfztdk32xmo7psizye3s46ena6f7yacikqvyx2u43b35cq",
            ])
            "state" = "PROVISIONING"
            "subnet_domain_name" = "workers03.clustervcn.oraclevcn.com"
            "time_created" = "2021-02-16 10:55:24.574 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
            "virtual_router_ip" = "10.0.12.1"
            "virtual_router_mac" = "00:00:17:CA:BC:FE"
        },
        {
            "availability_domain" = "LxbT:AP-MUMBAI-1-AD-1"
            "cidr_block" = "10.0.10.0/24"
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:23.559Z"
            })
            "dhcp_options_id" = "ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja"
            "display_name" = "Cluster-WorkerSubnet01"
            "dns_label" = "workers01"
            "freeform_tags" = tomap({})
            "id" = "ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaamoj23v524jprabzpoqb2fwsz5f7hfn7d7sm3tx53e5ah3w2fbigq"
            "ipv6cidr_block" = ""
            "ipv6public_cidr_block" = ""
            "ipv6virtual_router_ip" = ""
            "prohibit_public_ip_on_vnic" = false
            "route_table_id" = "ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq"
            "security_list_ids" = tolist([
            "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaavyp2tgcfztdk32xmo7psizye3s46ena6f7yacikqvyx2u43b35cq",
            ])
            "state" = "AVAILABLE"
            "subnet_domain_name" = "workers01.clustervcn.oraclevcn.com"
            "time_created" = "2021-02-16 10:55:23.97 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
            "virtual_router_ip" = "10.0.10.1"
            "virtual_router_mac" = "00:00:17:CA:BC:FE"
        },
        {
            "availability_domain" = "LxbT:AP-MUMBAI-1-AD-1"
            "cidr_block" = "10.0.11.0/24"
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:23.507Z"
            })
            "dhcp_options_id" = "ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja"
            "display_name" = "Cluster-WorkerSubnet02"
            "dns_label" = "workers02"
            "freeform_tags" = tomap({})
            "id" = "ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaajmmocnjfattmmu6i73lbnxa4tzt2bmtgarxbsifkzy6tuw5bumka"
            "ipv6cidr_block" = ""
            "ipv6public_cidr_block" = ""
            "ipv6virtual_router_ip" = ""
            "prohibit_public_ip_on_vnic" = false
            "route_table_id" = "ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq"
            "security_list_ids" = tolist([
            "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaavyp2tgcfztdk32xmo7psizye3s46ena6f7yacikqvyx2u43b35cq",
            ])
            "state" = "AVAILABLE"
            "subnet_domain_name" = "workers02.clustervcn.oraclevcn.com"
            "time_created" = "2021-02-16 10:55:23.76 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
            "virtual_router_ip" = "10.0.11.1"
            "virtual_router_mac" = "00:00:17:CA:BC:FE"
        },
        {
            "availability_domain" = "LxbT:AP-MUMBAI-1-AD-1"
            "cidr_block" = "10.0.21.0/24"
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:23.198Z"
            })
            "dhcp_options_id" = "ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja"
            "display_name" = "Cluster-LB-Subnet02"
            "dns_label" = "lb02"
            "freeform_tags" = tomap({})
            "id" = "ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaavrzf3cvidoe7cyt7nc5y5ahe2tk54rqtnss2ydzrjmeybkmgcdvq"
            "ipv6cidr_block" = ""
            "ipv6public_cidr_block" = ""
            "ipv6virtual_router_ip" = ""
            "prohibit_public_ip_on_vnic" = false
            "route_table_id" = "ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq"
            "security_list_ids" = tolist([
            "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaajynlozl5x27qrl4xdu63kgg5dxbpp7fvym6x2kxukimayk6fpjga",
            ])
            "state" = "AVAILABLE"
            "subnet_domain_name" = "lb02.clustervcn.oraclevcn.com"
            "time_created" = "2021-02-16 10:55:23.534 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
            "virtual_router_ip" = "10.0.21.1"
            "virtual_router_mac" = "00:00:17:CA:BC:FE"
        },
        {
            "availability_domain" = "LxbT:AP-MUMBAI-1-AD-1"
            "cidr_block" = "10.0.20.0/24"
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:23.191Z"
            })
            "dhcp_options_id" = "ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja"
            "display_name" = "Cluster-LB-Subnet01"
            "dns_label" = "lb01"
            "freeform_tags" = tomap({})
            "id" = "ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaabbs5k5jze7tm7vkqrllikss4eaixxx4yzwzlomhodf3c2iqzpdgq"
            "ipv6cidr_block" = ""
            "ipv6public_cidr_block" = ""
            "ipv6virtual_router_ip" = ""
            "prohibit_public_ip_on_vnic" = false
            "route_table_id" = "ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq"
            "security_list_ids" = tolist([
            "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaajynlozl5x27qrl4xdu63kgg5dxbpp7fvym6x2kxukimayk6fpjga",
            ])
            "state" = "AVAILABLE"
            "subnet_domain_name" = "lb01.clustervcn.oraclevcn.com"
            "time_created" = "2021-02-16 10:55:23.252 +0000 UTC"
            "vcn_id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
            "virtual_router_ip" = "10.0.20.1"
            "virtual_router_mac" = "00:00:17:CA:BC:FE"
        },
        ])
        VCN = tolist([
        {
            "cidr_block" = "10.0.0.0/16"
            "cidr_blocks" = tolist([
            "10.0.0.0/16",
            ])
            "compartment_id" = "ocid1.compartment.oc1..aaaaaaaa4cffiat2knvzy4bjs7eprhns5qspej5pfcelkfvrb7s5vahgr4yq"
            "default_dhcp_options_id" = "ocid1.dhcpoptions.oc1.ap-mumbai-1.aaaaaaaa5okqnbfsubcihc7yzudxpyrdersvwpdt7qtxsmxkukmtlg3crnja"
            "default_route_table_id" = "ocid1.routetable.oc1.ap-mumbai-1.aaaaaaaaryoa6tckuvufbl7dbwen55qnex6dklvrycy7nyrzr7haqqsgbzeq"
            "default_security_list_id" = "ocid1.securitylist.oc1.ap-mumbai-1.aaaaaaaamiik64dj5xmz7vwup37ubbomnjoiwq6vicn2yuucne4zkbt6ckza"
            "defined_tags" = tomap({
            "Oracle-Tags.CreatedBy" = "oracleidentitycloudservice/krishna"
            "Oracle-Tags.CreatedOn" = "2021-02-16T10:55:21.958Z"
            })
            "display_name" = "Cluster_vcn"
            "dns_label" = "clustervcn"
            "freeform_tags" = tomap({})
            "id" = "ocid1.vcn.oc1.ap-mumbai-1.amaaaaaaulymzmiabyun5tgtxeiqr7nezopo255kaz3yvpiqr6mmafaicb6q"
            "ipv6cidr_block" = ""
            "ipv6public_cidr_block" = ""
            "is_ipv6enabled" = false
            "state" = "AVAILABLE"
            "time_created" = "2021-02-16 10:55:22.064 +0000 UTC"
            "vcn_domain_name" = "clustervcn.oraclevcn.com"
        },
        ])
        cluster_id = "ocid1.cluster.oc1.ap-mumbai-1.aaaaaaaaae4dooddmfqtezbyg4zwkzjzg44tqy3fg44tkndbhcqtgzbzhezg"

Terraform plan is completed successfully. We can validate it by logging into cloud portal.

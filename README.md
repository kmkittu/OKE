# Provision Kubernetes Cluster With Terraform

In this workshop we develop Terraform script to setup Kubernetes Cluster.

## Steps
1) Collect the details of cloud attributes
2) Download the terraform script and extract it
3) Check the integrity of the code 
4) Execute the terraform code using Apply command

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

#### Fingerprint   
You can get the key's fingerprint with the following OpenSSL command. If you're using Windows, you'll need to install Git Bash for Windows and run the command with that tool.

    openssl rsa -pubout -outform DER -in ~/.oci/oci_api_key.pem | openssl md5 -c

Also in other way when you upload the public key in the Console (In the user details page) , the fingerprint is also automatically displayed there. It looks something like this: 12:34:56:78:90:ab:cd:ef:12:34:56:78:90:ab:cd:ef

![Fingerprint](https://github.com/kmkittu/OKE/blob/main/Add%20public%20Key%20-%20Fingerprint.png)

#### Region
Region at which OCI account is associated. You can find this information in the console easily

#### Compartment OCID
Open the navigation menu, under Identity you can find compartments. Click that. It will list all the compartments available with the account. Choose the compartment on which we need to create instance. That will show the compartment details including OCID as shown in the picture.

![Compartment](https://github.com/kmkittu/OKE/blob/main/Compartment%20OCID.png)



# Install caprover to an always free Oracle Compute Instance
These are ansible scripts to automate most of the tedius things needed to get a caprover instance up on a free oracle compute instance.

# DISCLAIMER: This does not work flawlessly anymore, lots of things have to be edited to function.

I have instructions for getting ansible and dependencies up on a Mac running Big Sur as well.

## Get account and API setup
First create a free account at https://cloud.oracle.com/

Once done, goto Profile - User Settings

Add Api Key

Generate Key and save the downloads to a new folder ~/.oci

Adjust security on private file

    chmod 600 /.oci/oracleidentitycloudservice_XXXXX-04-14-10-33.pem

Copy config and paste to ~/.oci/config

Change key_file line in config to private key file path:
key_file=~/.oci/oracleidentitycloudservice_XXXXX-04-14-10-33.pem

## Prepare deploy on Mac

    brew install pipenv oci-cli npm
    git clone https://github.com/axlroden/caprover-oci.git
    cd caprover-oci
    pipenv install
    pipenv shell
    ansible-galaxy collection install oracle.oci
    ansible-galaxy collection install community.docker
    ansible-galaxy collection install community.general
    exit

Get image OCID for your region

I use Ubuntu 18.04 minimal because caprover suggest not using 20.04.

https://docs.oracle.com/iaas/images/ubuntu-1804/

ocid1.image.oc1.eu-frankfurt-1.aaaaaaaaqto22mifcsemfb6soqca47ufyqread73zbz44ct7kra5csgy7hka

## Create .env file
    # Take from ~/.oci/config file
    COMPARTMENT_OCID=ocid1.tenancy.oc1..aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    # Adjust for your region
    IMAGE_OCID=ocid1.image.oc1.eu-frankfurt-1.aaaaaaaarfaszep3y25g6loj7hjgnhrlfz5w7zsjocpj4rvdgp6neyu4nuiq
    # This is your SSH key for the caprover instance
    PUBLIC_KEY=/Users/XXXXX/.ssh/id_rsa.pub

# Ansible magic
I have used Oracle's sample and changed / modified heavily.

https://github.com/oracle/oci-ansible-collection/tree/master/samples/compute/always_free_launch_compute_instance

## provision to oracle cloud
    pipenv run ansible-playbook provision.yaml

## install caprover on instance
    pipenv run ansible-playbook install.yaml

# Caprover
Once finished, verify you can reach your new caprover instance on http://public_ip:3000

Check instance.yaml (which gets generated during provision) for the IP if you are in doubt.

Create a wildcard A record for your domain, like *.cap.example.org, at your dns provider.

Once the DNS is propegated, finish the installation.

    npm i -g caprover
    caprover serversetup

Follow the instructions and Letsencrypt certs, passwords etc will be created and you can reach your new caprover instance.

# Destroy
If you want to destroy your cloud instance, I have included a destroy.yaml

It will remove the compute instance and anything attached to it.

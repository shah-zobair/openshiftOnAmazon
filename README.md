# openshiftOnAmazon
====================================================
https://github.com/rht-labs/openshift-ansible/blob/master/README_AWS.md
http://developers.redhat.com/blog/2016/08/22/build-your-next-cloud-based-paas-in-under-an-hour/
http://stackoverflow.com/questions/32297456/how-to-ignore-ansible-ssh-authenticity-checking
Always update the cluserid , before re-running and also change subnet-3rd octat.
====================================================

```
cd ~/.ssh
ssh-add ec2-EmeryAccount.pem
ssh-add -l
ansible --version
brew prune
brew upgrade
brew install python
pip install -U pyopenssl boto
```
##Check your ansible cfg file to ensure host_key_checking = False

```
anurags-MBP-2:~ anuragsaran$ cat ~/.ansible.cfg 
# config file for ansible -- http://ansible.com/
# ==============================================

# This config file provides examples for running
# the OpenShift playbooks with the provided
# inventory scripts. Only global defaults are
# left uncommented

[defaults]
# Add the roles directory to the roles path
roles_path = roles/

# Set the log_path
log_path = /tmp/ansible.log

forks = 10
host_key_checking = False
nocows = 1
# Need to handle:
# inventory - derive from OO_ANSIBLE_DIRECTORY env var
# callback_plugins - derive from pkg_resource.resource_filename
# private_key_file - prompt if missing
# remote_tmp - set if provided by user (cli)
# ssh_args - set if provided by user (cli)
control_path=/tmp

[ssh_connection]
control_path = %(directory)s/%%h-%%r
```

## create the config file so that you dont have to provide user and pem file.

```
anurags-MBP-2:.ssh anuragsaran$ cat config 
Host *.compute-1.amazonaws.com
  IdentityFile /Users/anuragsaran/.ssh/ec2-EmeryAccount.pem
  User = ec2-user

Host *.tryopenshift.online
  User = ec2-user
  IdentityFile /Users/anuragsaran/.ssh/ec2-EmeryAccount.pem
```

## Now try to ssh aws node and it should work
```
>> ssh aws-dns-name

>> anurags-MBP-2:ose-amazon anuragsaran$ git clone https://github.com/rht-labs/openshift-ansible
>> export AWS_ACCESS_KEY_ID=
>> export AWS_SECRET_ACCESS_KEY=
```

## Get the subnet and the cider for the subnet from AWS.
* subnet-3659ac6d
* 172.31.16.0/20

You possess a key pair that is imported in AWS and configured with your ssh-agent (ssh-add -l ).
The ssh key-pair is set up in your environment so you can authenticate without providing a password, i.e.
Master nodes will go to 21, 22, Infra Nodes:60 ETCD:40

```
>> cd /Users/anuragsaran/Documents/MW/os-demos/openshift-ansible
>> ansible-playbook  \
-i inventory/aws/hosts \
-e \
'num_masters=1 
num_nodes=2 
cluster_id=oscid 
cluster_env=dev
num_etcd=0 
num_infra=1 
deployment_type=openshift-enterprise 
private_subnet=172.31.16 
rhsm_username=asaran@redhat.com 
rhsm_password=
rhsm_pool=8a85f9874011071c01407da00b997cb2
zone=tryopenshift.online 
openshift_persist_registry=true
cli_ec2_infra_instance_type=t2.medium
cli_ec2_master_instance_type=t2.medium
cli_ec2_node_instance_type=m4.large
cli_ec2_vpc_subnet=subnet-3659ac6d
cli_ec2_keypair=ec2-EmeryAccount
cli_ec2_security_groups=openshiftSG 
cli_ec2_image=ami-2051294a 
cli_os_docker_vol_size=50 
cli_os_docker_vol_ephemeral=false 
cli_ec2_region=us-east-1' \
playbooks/aws/openshift-cluster/launch.yml
```
## Terminate the environment
```
>> ansible-playbook  \
-i inventory/aws/hosts \
-e \
'cluster_id=oscid
cluster_env=dev
deployment_type=openshift-enterprise 
zone=tryopenshift.online' \
playbooks/aws/openshift-cluster/terminate.yml
```


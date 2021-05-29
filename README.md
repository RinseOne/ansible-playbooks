# Ansible

## Base setup

### Prerequisites

* AWS account access configured
* ansible >= 2.9
* AWS and Posix collections from ansible galaxy
```
ansible-galaxy collection install amazon.aws
ansible-galaxy collection install ansible.posix
```

### VPC

This creates the VPC and security groups in the specified region.

```
ansible-playbook -e "aws_region=us-east-1" aws_vpc.yml
```

### Base AMI

Starts an instance from latest Ubuntu minimal AMI, customizes, creates the base AMI and terminates
the instance. It also cleans up old AMIs, keeping only the latest one.

```
ansible-playbook -e "aws_region=us-east-1" create_base_ami.yml
```

### Web Base AMI

Starts an instance from the base AMI, customizes, creates the web base AMI and terminates the
instance. It also cleans up old web base AMIs, keeping only the latest one.

```
ansible-playbook -e "aws_region=us-east-1" create_web_base_ami.yml
```

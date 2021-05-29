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

## Releases

### Web

Starts an instance from the web base AMI, installs specified `rinseweb` release as a daemon,
creates the AMI and terminates the instance. It also cleans up old web app AMIs, keeping only the
latest two.

```
ansible-playbook -e "aws_region=us-east-1" -e "web_app_release_version=VERSION" create_web_app_ami.yml
```

Just substitute the `VERSION` with the actual version; for example

```
ansible-playbook -e "aws_region=us-east-1" -e "web_app_release_version=0.0.1" create_web_app_ami.yml
```

This command will expect the OTP release archive to be available at the following location

```
https://github.com/RinseOne/rinseweb/releases/download/0.0.1/rinseweb.tar.gz
```

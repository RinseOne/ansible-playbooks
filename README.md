# Ansible Playbooks

- [Ansible Playbooks](#ansible-playbooks)
  - [Base setup](#base-setup)
    - [Prerequisites](#prerequisites)
    - [VPC](#vpc)
    - [Base AMI](#base-ami)
    - [Provision Bastion Host](#provision-bastion-host)
    - [Web Base AMI](#web-base-ami)
    - [Web App AMI](#web-app-ami)
    - [Load Balancer and Autoscaling Group](#load-balancer-and-autoscaling-group)
  - [Deployments](#deployments)
    - [Create and Deploy a New Web App AMI](#create-and-deploy-a-new-web-app-ami)
  - [SSH to Bastion and beyond](#ssh-to-bastion-and-beyond)

## Base setup

This section covers one-time base infrastructure setup.

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

### Provision Bastion Host

Uses the base AMI to create a bastion host.

```
ansible-playbook -e "aws_region=us-east-1" provision_bastion.yml
```

### Web Base AMI

Starts an instance from the base AMI, customizes, creates the web base AMI and terminates the
instance. It also cleans up old web base AMIs, keeping only the latest one.

```
ansible-playbook -e "aws_region=us-east-1" create_web_base_ami.yml
```

### Web App AMI

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
https://github.com/RinseOne/rinseweb/releases/download/0.0.1/rinseweb-ubuntu-22.04-amd64.tar.gz
```

Optionally, when `-e "pause_after_instance_ready=true"` flag is specified, the playbook will pause after the web app
candidate instance is ready, before taking its AMI and shutting it down.

### Load Balancer and Autoscaling Group

Initial setup creates the load balancer and the autoscaling group without the HTTPS listener.

```
ansible-playbook -e "aws_region=us-east-1" -e "initial=true" deploy_web_app_ami.yml
```

Create the hosted zone in AWS Route53 - we pass the `initial=true` flag to avoid creating a cert yet.
This will also point (alias) the domain to the ALB, so AWS can tell it is hosting it.

```
ansible-playbook -e "domain=rinse.one" -e "initial=true" aws_web_domain_and_cert.yml
```

After this is completed, update the domain's DNS servers to those listed in Route53 hosted zone.
Once this has taken effect, then we can proceed with the next step to create the SSL certificate.

Re-run the same playbook without the `initial` flag to create the auto-renewing certificate.

```
ansible-playbook -e "domain=rinse.one" aws_web_domain_and_cert.yml
```

Finally, create the HTTPS listener for the ALB that uses the SSL certificate just created.

```
ansible-playbook -e "aws_region=us-east-1" deploy_web_app_ami.yml
```

## Deployments

This section covers steps needed every time we want to deploy a new version of the web app.

### Create and Deploy a New Web App AMI

The single playbook accomplishes the task. It performs the following steps:

* Starts an EC2 instance using the latest web base AMI
* Downloads, extracts and installs the specified release archive to it as a systemd service
* Creates a new AMI
* Cleans up old AMIs, keeping the last 2, and shuts down the instance
* Uses the newly made AMI to deploy to the autoscaling group

```
ansible-playbook -e "aws_region=us-east-1" -e "web_app_release_version=VERSION" create_and_deploy_web_app_ami.yml
```

Similarly to above, substitute `VERSION` with the actual version.

Optionally, when `-e "pause_after_instance_ready=true"` flag is specified, the playbook will pause after the web app
candidate instance is ready, before taking its AMI and shutting it down.

## SSH to Bastion and beyond

Prerequiesite: `ssh-agent` must be running on local machine and it must have the keys added to it via `ssh-add`.

External SSH sessions can only be established to the bastion host. Get the IP address of the bastion host from the AWS console, and then

```
ssh -A ubuntu@<ip address>
```

The `-A` option enables forwarding of connections from ssh-agent, so we can now SSH from bastion to any other host using its internal IP address.

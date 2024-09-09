# FileFlux Ansible Automation

## Overview
This project contains Ansible playbooks to automate the management of ZFS pools and EBS volumes associated with the self-managed AWS EKS worker nodes. The playbooks handle tasks such as discovering EC2 instances based on tags, connecting through a bastion host, and running scripts on the EKS worker node to create new EBS volumes, attaching it to the appropriate worker node, adding it to the ZFS pool, and resizing the ZFS pool. The automation is designed to be used in the FileFlux project, which uses ZFS pools across various Kubernetes worker nodes to store end-user files. The project assumes that you have the FileFlux worker node AMI already built which includes the necessary script to add EBS volumes to the existing ZFS pool. Furthermore, it also assumes that you already have the FileFlux EKS cluster set up and running along with the bastion host and have access to the bastion host and EKS worker node key pairs.

## Features
- Discover and manage EKS worker nodes in AWS.
- Deploy and run scripts on the EKS worker nodes via a bastion host.
- Automate EBS volume creation, attachment, and ZFS pool resizing using Ansible playbooks.
- Interacts with FileFlux Worker to perform various actions via various Kubernetes services.
- Prompts for essential variables like bastion host IP, private keys, and EBS volume size.

## Prerequisites
To run this application locally or in a container, you need:
- Ansible installed locally [guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- AWS credentials configured to access EKS worker nodes [guide](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)
- AWS Infrastructure setup with EKS cluster, bastion host, and worker nodes located [here](https://github.com/fileflux/infra)

## Repository Structure

```plaintext
ansible/             
├── aws_ec2.yaml              
├── README.md               
├── deploy.yaml
```

### What Each File Does
- **aws_ec2.yaml**: This playbook uses the amazon.aws.aws_ec2 plugin to discover EC2 instances in the us-east-1 region based on tags, specifically 'karpenter.sh/discovery: s3'. It helps dynamically manage EKS worker nodes within your infrastructure.

- **deploy.yaml**: This playbook connects to private EKS worker nodes via a bastion host and runs the ZFS configuration script (zfs_add_script.sh). It prompts for the bastion host IP, private keys, and the EBS volume size, then executes the script using the provided details.

## Running the Playbooks

To discover EKS worker nodes dynamically and run the deploy.yaml Ansible playbook to add EBS volumes to the ZFS pool on all EKS worker nodes via a bastion host, use the following command:
   ```bash
   ansible-playbook -i aws_ec2.yaml deploy.yaml
   ```

The playbook will prompt for:

- The bastion host IP
- The path to the bastion host's private key (geneated during the AWS infra setup)
- The path to the EKS worker node's private key (geneated during the EKS cluster setup)
- The size of the EBS volume to be added to the ZFS pool

## Additional Notes
- Ensure that the zfs_add_script.sh is present on the EKS worker nodes under /home/ubuntu/ before running the deploy.yaml playbook. If you're creating AWS infrastructure using the FileFlux Terraform scripts, the zfs_add_script.sh will be copied to the EKS worker nodes automatically and you don't need to worry about it.

- The playbook uses ansible.builtin.expect to handle interactive input during the execution of the ZFS script, such as entering the EBS volume size.
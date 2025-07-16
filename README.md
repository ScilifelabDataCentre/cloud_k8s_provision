# Kubernetes on OpenStack with Terraform and Ansible

This project automates the deployment of a Kubernetes cluster on OpenStack using Terraform for infrastructure provisioning and Ansible for configuration management.

## Overview

The goal of this repository is to provide a simple and reproducible way to set up a Kubernetes cluster.
- **Terraform** is used to create the required infrastructure on OpenStack, including virtual machines, networking, and security groups.
- **Ansible** is used to install and configure Kubernetes on the provisioned instances, setting up a master node and one or more worker nodes.

## Architecture

The deployment process consists of two main phases:

1.  **Infrastructure Provisioning:** Terraform reads the `.tf` files to build the necessary OpenStack resources. This includes:
    *   A virtual network with a subnet and a router connected to the external network.
    *   A security group with rules to allow necessary traffic (SSH, Kubernetes API, etc.).
    *   An SSH keypair for accessing the instances.
    *   Virtual machine instances for the Kubernetes master and worker nodes.
    *   An Ansible inventory file is dynamically generated with the details of the created instances.

2.  **Kubernetes Installation:** After the infrastructure is ready, an Ansible playbook (`playbooks/install.yml`) is run to:
    *   Install container runtime (e.g., containerd), `kubeadm`, `kubelet`, and `kubectl` on all nodes.
    *   Initialize the Kubernetes master node using `kubeadm`.
    *   Join the worker nodes to the cluster.
    *   Install necessary tools like Helm.

## Directory Structure

- `bin/`: Contains helper scripts.
- `bootstrap/`: Shell scripts for bootstrapping master and worker nodes, called by Ansible.
- `common/inventory/`: Terraform module to generate the Ansible inventory file.
- `keypair/`: Terraform module for managing SSH keypairs.
- `network/`: Terraform module for creating the network stack.
- `node/`: Terraform module for creating master and worker nodes.
- `playbooks/`: Contains the Ansible playbooks for configuration.
- `secgroup/`: Terraform module for managing security groups.
- `templates/`: Contains template files, like the `tfvars` template.
- `main.tf`: The main Terraform file that orchestrates the infrastructure deployment.
- `ansible.cfg`: Ansible configuration file.

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) (>= 0.14.0)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- An OpenStack account with credentials.
- An SSH key pair.

## Deployment Steps

1.  **Configure OpenStack Credentials**

    You need to source your OpenStack `openrc.sh` file to authenticate Terraform and Ansible with your cloud.

    ```bash
    source your-openrc.sh
    ```

2.  **Prepare Terraform Variables**

    Create a `terraform.tfvars` file from the template.

    ```bash
    cp templates/config.tfvars.openstack-template terraform.tfvars
    ```

    Now, edit `terraform.tfvars` and fill in the required values for your OpenStack environment (e.g., `external_network_uuid`, `boot_image`, flavors, etc.). You should also specify the path to your public SSH key.

    ```hcl
    # Example terraform.tfvars
    cluster_prefix        = "my-k8s-cluster"
    ssh_key               = "~/.ssh/id_rsa.pub"
    external_network_uuid = "your-external-network-uuid"
    boot_image            = "Ubuntu 20.04"
    master_flavor         = "m1.medium"
    worker_flavor         = "m1.medium"
    worker_count          = 2
    floating_ip_pool      = "public"
    kubeadm_token         = "your-secure-token"
    ```

3.  **Provision Infrastructure with Terraform**

    Initialize Terraform, review the plan, and apply it.

    ```bash
    terraform init
    terraform plan
    terraform apply
    ```

    This will create all the necessary resources and generate an `inventory` file for Ansible.

4.  **Install Kubernetes with Ansible**

    Once Terraform has finished, run the Ansible playbook to configure the nodes and install Kubernetes.

    ```bash
    ansible-playbook -i inventory playbooks/install.yml
    ```

5.  **Access Your Cluster**

    The playbook should place a `kubeconfig` file in the root directory. You can use this to interact with your cluster.

    ```bash
    export KUBECONFIG=$(pwd)/kubeconfig
    kubectl get nodes
    ```

## Cleanup

To destroy all the resources created by this project, run the following command:

```bash
terraform destroy
``` 
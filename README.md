# Ansible Playbook to deploy Ika Validator Node 

This Ansible playbook automates the deployment and configuration of Ika Validator Node. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as validator and node keys. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves private validator keys and node keys from HashiCorp Vault. The keys are expected to follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`
For example:
`testnet/ika/encapsulate/validator/network.key`

This structure ensures easy organization and secure retrieval of secrets.

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/ika-ansible.git
cd ika-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for `testnet.yml`

```yaml
---
all:
  vars:
    env: testnet
  children:
    ika:
      hosts:
        validator.ika.testnet.encapsulate.xyz:
          type: validator
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the port, source url configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version specific variables.
- **`group_vars/vault.yml`**: Store secret variables, such as `jwtsecret`, in this file.
- There are role specific variables defined in each roles `default/main.yml` and `vars/main.yml`.

**Note**: Make sure to set the `VAULT_TOKEN` environment variable, as it enables logging in and fetching secrets from HashiCorp Vault.

Create a `group_vars/vault.yml` with your preferred ansible-vault password:

```
ansible-vault create group_vars/vault.yml
```

Example `group_vars/vault.yml`:

```
# maintains anything sensitive like api keys
vault:
  default:
    hcp:
      vault:
        url:
  mainnet:
    sui:
      rpc_url:
    external:
      ika_package_id:
      ika_system_package_id:
      ika_system_object_id:
      ika_common_package_id:
      ika_dwallet_2pc_mpc_package_id:
      ika_dwallet_coordinator_object_id :
  testnet:
    sui:
      rpc_url:
    external:
      ika_package_id:
      ika_system_package_id:
      ika_system_object_id:
      ika_common_package_id:
      ika_dwallet_2pc_mpc_package_id:
      ika_dwallet_coordinator_object_id :
```

### Usage

1. First, install the dependencies:

  ```
  ansible-galaxy collection install -r collections/requirements.yml
  ```

2. Create a `ansible_vault_password` file containing ansible-vault password

3. Configure your remote server username and private key file path in the `ansible.cfg` file. Additionally, set the SSH port for your server by adjusting the `ansible_port` variable in `group_vars/all.yml`.

4. Then run the playbook:

- To deploy validator or fullnode:

```
ansible-playbook setup_node.yml -l validator.ika.testnet.encapsulate.xyz -e "fetch_validator_keys=true"
```

- To deploy bridge node:

```bash
ansible-playbook setup_bridge.yml -l bridge.ika.testnet.encapsulate.xyz -e "fetch_validator_keys=true"
```

**Note**: The default value for `fetch_validator_keys` is false, which disables fetching keys from Hashicorp Vault.

After you run the playbook, it will ask for confirmation, displaying all the variables and the IP address or DNS of the server you are going to deploy.

Example output:

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [validator.ika.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.ika.testnet.encapsulate.xyz",
        "Groups: ['ika']",
        "Project: ika",
        "Environment: testnet",
        "Type: validator",
        "Version: v1.0.0",
        "Service Name: ika",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [validator.ika.testnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [validator.ika.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.ika.testnet.encapsulate.xyz",
        "Project: ika",
        "Environment: testnet",
        "Type: validator"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [validator.ika.testnet.encapsulate.xyz]
```

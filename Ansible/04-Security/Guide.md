# Security

**Security should be the first or one of the first thing you consider. Not in step(Lab) #4.**

In this section you will:

- Create a secret in an existing Azure Key Vault.
- Modify your playbook so that you reference the secret in your playbook.

## Add a secret to Azure Key Vault

Azure Key Vault is a tool for securely storing and accessing secrets.

1. Use [azure_rm_keyvault](https://docs.ansible.com/ansible/latest/modules/azure_rm_keyvault_module.html) to create an Azure Key Vault. For the purpose of this lab, we have pre-provisioned an Azure Key Vault for each lab participant.
1. Use [azure_rm_keyvaultsecret](https://docs.ansible.com/ansible/latest/modules/azure_rm_keyvaultsecret_module.html) to add a secret. 

#### Cheat Sheet: create secret
<details>
<summary>
Expand to see how you can create a key vault secret
</summary>

```yml
  tasks:
  - name: create a Key Vault secret
    azure_rm_keyvaultsecret:
      keyvault_uri: "https://{{ keyvault_name }}.vault.azure.net"
      secret_name: "{{ secret_name }}"
      secret_value: "Password1234!"
```

</details>

## Retrieve secret from Azure Key Vault

While preparing for this workshop, we discovered we do not have a module for retrieving secret from Azure Key Vault.

We will be adding a new module called azure_rm_keyvaultsecret_info in future release. For the purpose of this lab, we will embed the new module using the `roles` keyword. (Roles will be explained in Lab 5.)

1. You will perform an extra step in this lab to copy the preview version of azure_rm_keyvaultsecret_info to your environment. Copy `azure_rm_keyvaultsecret_info.py` in `modules/library`(code/modules/library) to `clouddrive/ansible-playbooks/modules/library`. Make sure you keep the same folder structure `modules/library`
2. In the header, add:

```yml
  roles:
    - ./modules
```

3. Use azure_rm_keyvaultsecret_info to get secret you added in previous step. Here are a couple of examples on how to you can get secret.

```yml

  - name: Get latest version of a secret
    azure_rm_keyvaultsecret_info::
      vault_uri: "https://myVault.vault.azure.net"
      name: mysecret

  - name: Get a specific version of a secret
    azure_rm_keyvaultsecret_info:
      vault_uri: "https://myVault.vault.azure.net"
      name: mysecret
      version: 12345
```

### Cheat Sheet: get secret
<details>
<summary>
Expand to see how you can get a key vault secret
</summary>

```yml
  - name: Get latest version of a secret
    azure_rm_keyvaultsecret_info:
      vault_uri: "https://{{ keyvault_name }}.vault.azure.net"
      name: "{{ secret_name }}"
    register: output

- debug:
      var: output.secret.value
```

</details>

> **CODE**: To view all of the completed codes, go to [lab4-keyvault.yml](Code/lab4-keyvault.yml). 

## Replace hard coded password in previous lab

Next, instead of hard coding the password and exposing a major security risk when provisioning the VM, you can now:

1. Make sure you have already copied `azure_rm_keyvaultsecret_info.py` in `modules/library`(/modules/library) to `clouddrive/ansible-playbooks/modules/library` 
2. Run a task to retrieve the secret from Azure Key Vault. **Hint**: use `register output`
3. Use the value retrieved back from key vault to configure the admin password for the VM. **Hint**: use `output.secret.value`

### Cheat Sheet: pass secret retrieved from Key Vault to the next task

<details>
<summary>
Expand to see how you can pass secret retrieved from Key Vault to the next task
</summary>

```yml
  - name: Get latest version of a secret
    azure_rm_keyvaultsecret_info:
      vault_uri: "https://{{ keyvault_name }}.vault.azure.net"
      name: "{{ secret_name }}"
    register: output

  - name: Create a virtual machines
    azure_rm_virtualmachine:
      resource_group: "{{ myResource_group }}"
      name: "{{ myVM }}"
      admin_username: "testuser"
      admin_password: " {{ output.secret.value }}"
      vm_size: Standard_B1ms
      network_interfaces: "{{ myNIC }}"
      image:
        offer: UbuntuServer
        publisher: Canonical
        sku: 16.04-LTS
        version: latest
```

</details>

> **CODE:** To view all of the completed codes, go to [lab4.yml](Code/lab4.yml).  

## Further readings

This [sample playbook](https://github.com/Azure-Samples/ansible-playbooks/blob/master/keyvault_create.yml) for a good walk-through of creating an Azure Key Vault.



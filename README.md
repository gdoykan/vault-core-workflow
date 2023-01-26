# vault-core-workflow demo

## Overview
This demo walks through a couple of the core workflows of Vault covering how it handles authentication, authorization, and secrets management from both an admin and consumer perspective. 

### Workflows
**1. Configure Vault as an admin:**
- Set up an Auth Method (userpass)
- Create users and policies
- Configure a secrets engine (KV-V2)
- Configure the AWS secrets engine to generate dynamic credentials

**2. Consuming secrets as a client:**
- Authenticate to Vault
- Show user permissions from configured policies
- Read/Write secrets to Vault
- Use Dynamic credentials to access AWS




## Prerequisites:
- Vault CLI
- Repo cloned


## Steps
First run an instance of Vault locally in dev mode:
```
vault server -dev
```

1. **Configure userpass auth method**
    1. Create 2 users: admin, dev
    
    ```bash
    vault auth enable userpass
    vault write auth/userpass/users/developer password=password
    vault write auth/userpass/users/admin password=password
    ```
    
2. **Configure an admin policy**
    1. Review the admin policy in text first
    2. Create admin policy and attach it to the Admin entity
    
    ```bash
    vault policy write admin admin-policy.hcl
    vault write auth/userpass/users/admin/policies policies=admin
    ```
    
3. Create a **developer_credentials policy** - read only
    
    ```bash
    vault policy write dev dev-policy.hcl
    vault write auth/userpass/users/developer/policies policies=dev
    ```
    
4. **Log into the Admin user** for further configuration of Vault so that we are not using the Root token
    
    ```bash
    vault login -method=userpass username=admin
    ```
    
5. **Write a secret to the KV-V2 secrets engine**
    1. first show that the KV secret engine is enabled
    
    ```bash
    vault kv put secret/aws username="aws_access_id" password=aws_secret_key
    ```
    

**Dev workflow: Consuming Secrets from Vault**

1. Log into Vault as dev
    
    ```bash
    vault login -method=userpass username=developer
    ```
    
2. D**emonstrate RBAC of the dev user**
    1. Read the secret
        
        ```bash
        vault kv get secret/aws
        ```
        
    2. Attempt to write a new secret (should fail)
        
        ```bash
        vault kv put secret/aws username="changed_user" password="changed_pass"
        ```
        
3. Read credentials from the UI console

**Dynamic Credentials Workflow:**

1. Log into Admin
    
    ```bash
    vault login -method=userpass username=admin
    ```
    
2. **configure the AWS Secrets Engine**
    1. Create a `developer` role with an `iam_user` policy with capability to read from the AWS path
    2. Look at the iam_user.json first in text
    3. Create a Vault role that maps to a set of AWS permissions and credentials type. So when a user generates credentials they are mapped to this role.
    
    **NOTE**: Have to add the `aws` path in your admin policy to be able to execute these commands
    
    ```bash
    vault secrets enable aws
    
    vault write aws/config/root \
        access_key={insert access key} \
        secret_key={insert secret key} \
        region=us-east-1
    
    vault write aws/roles/developer credential_type=iam_user policy_document=@iam_user.json
    ```
    
3. Log into the Dev user
    
    ```bash
    vault login -method=userpass username=developer
    ```
    
4. **Read the credentials** from AWS secrets engine
    
    ```bash
    vault read aws/creds/developer
    ```
    
5. Use the credentials to **log into AWS**
    - aws configure on CLI
    - Show that I am logged in

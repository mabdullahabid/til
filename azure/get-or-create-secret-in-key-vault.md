# Get or Create Secrets in Azure Key Vault using Python

A script to get or create secrets in Azure Key Vault using Python.

I was brought in to a project where a large Django monolith was being moved
from Heroku to Azure. It was decided that we'll be using Azure Kubernetes
Service (AKS) for hosting the Django application. The application had a lot of
environment variables (config variables on Heroku). We decided to use Azure Key
Vault where we injected multiple environment variables into a single secret
with an export command which were then sourced in the Kubernetes deployment.

It was super painful to manually create secrets in the Azure Key Vault and I
had not worked with kubectl before so I decided to write a Python script to help me.

Install Azure CLI and Python SDK for Azure:

```zsh
pip install azure-cli
pip install azure-identity
pip install azure-keyvault-secrets
```

Login to Azure:

```zsh
az login
```

Python script to get or create secrets in Azure Key Vault:

```python
import os
import re
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

class AzureSecretClient:
    def __new__(cls, key_vault_name):
        key_vault_uri = f"https://{key_vault_name}.vault.azure.net"
        credential = DefaultAzureCredential()
        client = SecretClient(vault_url=key_vault_uri, credential=credential)
        return client


class KeyVaultManipulator:
    def __init__(self, client):
        self.client = client
        
    def get_env_var_value(self, env_string, variable_name):
        # Define the regex pattern to find the exact environment variable and its value
        # It looks for the variable name followed by an equal sign and then captures everything between the quotes
        # \b around the variable_name ensures it matches as a whole word and not as part of another variable
        pattern = rf'\b{variable_name}\b="([^"]*)"'
        
        # Use the search function from the re module to find the first match
        match = re.search(pattern, env_string)
        
        # If a match is found, return the captured group (the value), otherwise return None
        env_var_value = match.group(1) if match else None
        if env_var_value:
            print(f"{variable_name}: {env_var_value}")
        return env_var_value
    
    def create_env_var(self, env_string, variable_name, new_value):
        # Check if the variable already exists
        current_value = self.get_env_var_value(env_string, variable_name)
        if current_value is None:
            print(f"Unable to find {variable_name} in envdata. Creating...")
            # If it does not exist, append the new variable in the correct format
            if env_string and not env_string.endswith('\n'):
                env_string += '\n'
            env_string += f'export {variable_name}="{new_value}"\n'
        else:
            print(f"Found {variable_name} in envdata. Updating from {current_value} to {new_value}")
            env_string = self.update_env_var(env_string, variable_name, new_value)
        
        return env_string
    
    
    def update_env_var(self, env_string, variable_name, new_value):
        # Define the regex pattern to find the exact environment variable with double quotes
        # \b around the variable_name ensures it matches as a whole word and not as part of another variable
        pattern = rf'(\b{variable_name}\b=)"[^"]*"'
        
        # Replace the old value with the new one using the sub function from the re module
        # The replacement string will reuse the matched first group (variable_name=") and put the new value in double quotes
        updated_env_string = re.sub(pattern, rf'\1"{new_value}"', env_string, count=1)
        
        return updated_env_string
        
    
    def delete_env_var(self, env_string, variable_name):
        # Define the regex pattern to match the full line containing the environment variable
        # \b around the variable_name ensures it matches as a whole word and not as part of another variable
        pattern = rf'^export \b{variable_name}\b="[^"]*"\n?'
        
        # Use the sub function from the re module to replace the matched line with an empty string
        updated_env_string = re.sub(pattern, '', env_string, flags=re.MULTILINE)
        
        return updated_env_string
    
    
    def count_env_var(self, env_string):
        # Define the regex pattern to match lines that start with 'export', followed by any characters, an equal sign, and quoted value
        pattern = r'export \w+="[^"]*"'
        
        # Use the findall function from the re module to find all matches in the string
        matches = re.findall(pattern, env_string)
        
        # The length of the list of matches will be the count of environment variables
        return len(matches)
    
    def get_envdata(self, env_key):
        # Retrieve environment data from all key vault secrets
        cbenvdata1 = self.client.get_secret("cbenvdata1")
        cbenvdata2 = self.client.get_secret("cbenvdata2")
        cbenvdata3 = self.client.get_secret("cbenvdata3")
    
        # Print counts for debugging
        print(f"len(cbenvdata1): {self.count_env_var(cbenvdata1.value)}")
        print(f"len(cbenvdata2): {self.count_env_var(cbenvdata2.value)}")
        print(f"len(cbenvdata3): {self.count_env_var(cbenvdata3.value)}")
    
        # Initialize envdata to None
        envdata = None
    
        # Check each cbenvdata's value to find an exact match using regex with word boundaries
        search_pattern = rf'\b{env_key}\b='
    
        if re.search(search_pattern, cbenvdata1.value):
            envdata = cbenvdata1
        elif re.search(search_pattern, cbenvdata2.value):
            envdata = cbenvdata2
        elif re.search(search_pattern, cbenvdata3.value):
            envdata = cbenvdata3
    
        # If envdata is still None, raise an error
        if not envdata:
            raise TypeError(f"Unable to find {env_key} in envdata.")
    
        # Print out where the key was found
        print(f"Key Vault: {envdata}")
        print(f"Found {env_key} in {envdata.name}")
    
        return envdata

    
    def set_envdata(self, envdata, env_key, env_value):
        # Use create_env_var to handle both creation and update of the variable
        updated_value = self.create_env_var(envdata.value, env_key, env_value)
        # Update the secret in Azure Key Vault
        return self.client.set_secret(envdata.name, updated_value)
    
    
    def unset_envdata(self, envdata, env_key):
        # Use delete_env_var to handle deletion of the variable
        updated_value = self.delete_env_var(envdata.value, env_key)
        # Update the secret in Azure Key Vault
        return self.client.set_secret(envdata.name, updated_value)
```

Usage

Get environment variable

```python
key_vault_list = [
    "key-vault-1",
    "key-vault-2",
]
for vault in key_vault_list:
    client = AzureSecretClient(vault)
    manipulator = KeyVaultManipulator(client=client)

    envvar_key = "ENV_VAR_KEY"

    print(vault)
    print("--------------------------")
    try:
        envdata = manipulator.get_envdata(envvar_key)
        envvar_val = manipulator.get_env_var_value(envdata.value, envvar_key)
    except TypeError as e:
        print(e)
    finally:
        print("--------------------------")
```

Set environment variable

```python
key_vault_list = [
    "key-vault-1",
    "key-vault-2",
]
for vault in key_vault_list:
    client = AzureSecretClient(vault)
    manipulator = KeyVaultManipulator(client=client)

    envvar_key = "ENV_VAR_KEY"

    print(vault)
    print("--------------------------")
    try:
        envdata = manipulator.get_envdata(envvar_key)
        manipulator.set_envdata(envdata, envvar_key, "ENV_VAR_VALUE")
    except TypeError as e:
        print(e)
    finally:
        print("--------------------------")
```

Unset environment variable

```python
key_vault_list = [
    "key-vault-1",
    "key-vault-2",
]
for vault in key_vault_list:
    client = AzureSecretClient(vault)
    manipulator = KeyVaultManipulator(client=client)

    envvar_key = "ENV_VAR_KEY"

    print(vault)
    print("--------------------------")
    try:
        envdata = manipulator.get_envdata(envvar_key)
        manipulator.unset_envdata(envdata, envvar_key)
    except TypeError as e:
        print(e)
    finally:
        print("--------------------------")
```
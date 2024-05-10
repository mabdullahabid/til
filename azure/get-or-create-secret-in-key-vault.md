# Get or Create Secrets in Azure Key Vault using Python

This needs to be updated to the latest version.

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

key_vault_name = "kv-testapp-dev-eastus-001"
key_vault_uri = f"https://{key_vault_name}.vault.azure.net"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=key_vault_uri, credential=credential)

def get_env_var_value(env_string, variable_name):
    # Define the regex pattern to find the exact environment variable and its value
    # It looks for the variable name followed by an equal sign and then captures everything between the quotes
    pattern = rf'{variable_name}="([^"]*)"'
    
    # Use the search function from the re module to find the first match
    match = re.search(pattern, env_string)
    
    # If a match is found, return the captured group (the value), otherwise return None
    return match.group(1) if match else None

def create_env_var(env_string, variable_name, new_value):
    # Check if the variable already exists
    current_value = get_env_var_value(env_string, variable_name)
    if current_value is None:
        print(f"Unable to find {variable_name} in envdata. Creating...")
        # If it does not exist, append the new variable in the correct format
        if env_string and not env_string.endswith('\n'):
            env_string += '\n'
        env_string += f'export {variable_name}="{new_value}"\n'
    else:
        print(f"Found {variable_name} in envdata. Updating from {current_value} to {new_value}")
        env_string = update_env_var(env_string, variable_name, new_value)
    
    return env_string


def update_env_var(env_string, variable_name, new_value):
    # Define the regex pattern to find the exact environment variable with double quotes
    pattern = rf'({variable_name}=)"[^"]*"'
    
    # Replace the old value with the new one using the sub function from the re module
    # The replacement string will reuse the matched first group (variable_name=") and put the new value in double quotes
    updated_env_string = re.sub(pattern, rf'\1"{new_value}"', env_string, count=1)
    
    return updated_env_string


def count_env_var(env_string):
    # Define the regex pattern to match lines that start with 'export', followed by any characters, an equal sign, and quoted value
    pattern = r'export \w+="[^"]*"'
    
    # Use the findall function from the re module to find all matches in the string
    matches = re.findall(pattern, env_string)
    
    # The length of the list of matches will be the count of environment variables
    return len(matches)

def get_envdata(env_key):
    testenvdata1 = client.get_secret("testenvdata1")
    testenvdata2 = client.get_secret("testenvdata2")
    testenvdata3 = client.get_secret("testenvdata3")
    
    print(count_env_var(testenvdata1.value))
    print(count_env_var(testenvdata2.value))
    print(count_env_var(testenvdata2.value))
    
    if env_key in testenvdata1.value:
        return testenvdata1
    elif env_key in testenvdata2.value:
        return testenvdata2
    elif env_key in testenvdata3.value:
        return testenvdata3

def set_envdata(envdata, env_key, env_value):
    # Use create_env_var to handle both creation and update of the variable
    updated_value = create_env_var(envdata.value, env_key, env_value)
    # Update the secret in Azure Key Vault
    return client.set_secret(envdata.name, updated_value)
```

Usage

Getting the environment data and value:
```python
envvar_key = "DEBUG"
envdata = get_envdata(envvar_key)
print(envdata)
envvar_val = get_env_var_value(envdata.value, envvar_key)
print(envvar_val)
```

Setting the environment data and value:
```python
set_envdata(envdata, envvar_key, "NEW_VALUE")
```

